# MiMo-V2.5 Research Deep-Dive

Consolidated learning document from 10 parallel research agents.
Model: **MiMo-V2.5** (and its siblings MiMo-V2-Flash, MiMo-V2.5-Pro) by **Xiaomi LLM-Core team**.
Primary sources: MiMo-V2-Flash Technical Report (arXiv:2601.02780v2, 31 pages), XiaomiMiMo HF model cards, Full-Pipeline Inference Optimization paper (arXiv:2607.13095v1).

---

## 0. TL;DR — the 60-second summary

MiMo-V2.5 is Xiaomi's open-source (MIT) **310B-parameter sparse MoE LLM (15B active) that is native-omnimodal** (text + image + video + audio) with a **1M-token context window**, trained on ~48T tokens. Its competitive edge comes from 5 stacked innovations:

1. **Hybrid Sliding-Window Attention (SWA)** — only 9 of 48 layers use full global attention; the rest attend within a 128-token window. Cuts KV-cache & attention compute ~6-7x. Works because of a **learnable attention-sink bias**.
2. **Sparse MoE** — 256 experts, 8 active per token. 310B total params but only 15B of compute per token.
3. **Multi-Token Prediction (MTP)** — the model predicts the next 3 tokens. Doubles as a native self-speculative-decoding draft model → ~2.6x decode speedup AND faster RL rollouts (the real scaling lever).
4. **Multi-Teacher On-Policy Distillation (MOPD)** — the post-training recipe. Train domain-specialist teacher models via RL, then distill them all into one student using the student's OWN rollouts with dense token-level rewards. Solves the "see-saw" capability tradeoff.
5. **Relentless efficiency engineering** — FP8 training, 1M context via progressive extension, and a production serving stack that turns the theoretical SWA savings into 7x cheaper serving.

The data-scientist lesson: **MiMo wins on efficiency-per-dollar, not raw intelligence.** It trades raw benchmark ceilings (where Gemini/Claude/GPT still lead) for "frontier-adjacent at 1/10th the cost" — the Pareto frontier of performance and efficiency.

---

## 1. Hybrid Sliding-Window Attention (SWA) — the core architectural bet

### 1.1 The interleaving pattern

- Model stacks **M=8 hybrid blocks**: each block = **5 SWA layers + 1 Global Attention (GA) layer** → 5:1 ratio. First transformer block is GA + dense FFN (stabilizes early representation learning).
- **MiMo-V2.5**: 48 layers = 39 SWA + 9 GA, window W=128. **MiMo-V2.5-Pro**: 70 layers = 60 SWA + 10 GA (6:1 ratio). SWA window is always **128 tokens**.
- Same trick in vision & audio encoders (see sections 5-6).

### 1.2 The learnable attention sink bias — THE key innovation

This is what makes aggressive SWA viable. Each attention head owns **one learnable scalar `sink`**.

- Softmax is computed over the window **plus a virtual "sink token"** with logit `sink`. If nothing in the window matches the query, the head dumps attention mass into the sink instead of spreading probability over noise.
- The sink is dead mass (no value), giving every head an **"abstain" option**. Heads can specialize: some consume local info, others switch off, GA layers handle global dependencies.
- One scalar per head = tens of floats total. Minimal parameter cost, massive robustness gain.

**Ablation evidence (32B dense model, 250B tokens):**

| Config | MMLU | BBH | MATH |
|---|---|---|---|
| All Global Attention | 57.3 | 54.7 | 9.5 |
| Hybrid W=128, **no sink** | 54.9 | 52.4 | 8.9 |
| Hybrid W=128, **with sink** | **58.3** | **56.1** | 10.3 |

The sink variant **beats full attention**. Long-context: hybrid+sink beats all-GA on GSM-Infinite (17.3 vs 12.3), NoLiMa, MRCR. Counter-intuitively, **W=512 is WORSE than W=128** — a tight window forces clean division of labor and acts as regularization.

### 1.3 GQA details (correction to what some sources claim)

- **Technical report is authoritative**: SWA has **64 Q heads / 8 KV heads**; GA has **64 Q / 4 KV heads**. (The HF V2.5 card annotation appears swapped.) Head dims: QK=192, V=128, partial RoPE on first 64 QK dims.
- Rationale: GA KV grows with full context N → compress harder (4 KV heads). SWA KV is bounded by window regardless → afford richer per-token KV (8 heads).
- V2.5-Pro: 128 Q heads, uniform 8 KV heads (16x compression).

### 1.4 Quantified wins

- KV-cache storage: ~1/5.3 (V2-Flash) to 1/7 (V2.5-Pro) of full attention → "6x-7x reduction."
- Attention compute: O(N²) → O(W·N) on SWA layers. Prefill is compute-bound → direct prefill cost cut. Decode is memory-bound → KV cut ≈ direct decode cost cut at long context.
- RoPE split: **GA base = 640K** (rises to 5M at 256K context), **SWA base = 10,000** (unchanged). SWA only attends within 128 positions so low base is fine; GA needs huge base to keep far positions distinguishable.

### 1.5 Long-context evidence

- NIAH-Multi retrieval: 99.3/99.9/98.6/96.7 at 32K/64K/128K/256K.
- GSM-Infinite: degrades 37.7→29.0 from 16K→128K (DeepSeek-V3.2 collapses 50.4→25.7, Kimi-K2 34.6→8.8).
- V2.5-Pro GraphWalks: 0.56 BFS at 512K, 0.37 at 1M — prior V2-Pro collapses to 0.00 at 1M.

### Data scientist intuition

Think of it as a **Pareto improvement in the memory/quality plane**. Full attention = O(N²) compute, O(N) KV/layer. Hybrid SWA makes both ~linear. The sink is like giving a softmax classifier a null category so it isn't forced to predict nonsense. SWA layers become conv-like local feature extractors; GA layers are sparse "teleporters." **Both prefill AND decode benefit** — rare, since most optimizations help one or the other. For 1M-token agent workloads this is the difference between "impossible to serve" and "cheap to serve."

---

## 2. Sparse Mixture-of-Experts (MoE)

### 2.1 Configuration

| | V2-Flash | V2.5 | V2.5-Pro |
|---|---|---|---|
| Total / active | 309B / 15B | 310B / 15B | 1.02T / 42B |
| Layers | 48 (1 dense + 47 MoE) | 48 | 70 |
| Hidden size | 4096 | 4096 | 6144 |
| Routed experts | 256 | 256 | 384 |
| Experts/token | 8 | 8 | 8 |
| MoE intermediate | 2048 | 2048 | 2048 |
| Dense FFN (layer 0 only) | 16384 | 16384 | 16384 |

**Key design fact**: 8 experts × 2048 intermediate = **16384 = exactly the dense FFN width**. Per-token FFN compute is identical across all layers (dense layer 0 = compute-equivalent of one MoE layer). This makes the architecture trivially resource-schedulable.

### 2.2 Routing & load balancing

- Plain top-k gating, **NO shared expert** (unlike DeepSeek-V3 which appends one).
- **Router parameters kept in FP32** (top-k selection is numerically delicate — FP8 rounding would flip selections).
- Pre-training load balancing: sequence-level auxiliary loss with coefficient **1e-5 (100x smaller than DeepSeek-V3's 0.001)** + additive expert-bias mechanism. MiMo leans on the bias mechanism, not the quality-hurting aux loss.
- Production serving: learned routing is so uniform that average **expert load factor ≈ 0.85** — NO inference-time balancing needed at all.

### 2.3 MoE × RL training detail — R3 Rollout Routing Replay (§4.6.1)

A genuinely novel infrastructure trick: MoE models suffer **inconsistent expert routing between rollout (SGLang) and training (Megatron) due to FP8 precision**. R3 records which experts were routed during rollout and **replays those exact experts during training** — critical for stable on-policy RL. Also: request-level prefix cache stores BOTH KV cache AND routed experts from prior turns.

### Data scientist intuition

**MoE = capacity without compute.** A dense 310B model spends 310B FLOPs per token. MiMo stores 310B across 256 specialists, forces each token through 8 → 15B-scale compute with 310B-scale knowledge. The router is a per-token 256-way classifier. The ~4-5% active/total ratio (15/309, 42/1020) is the industry's converged sweet spot. Risk: router collapse onto "superstar" experts = back to a dense model + dead weights. The 1e-5 aux loss + bias prevent this.

---

## 3. Multi-Token Prediction (MTP)

### 3.1 What it is

- Training objective where the model predicts the **next K=3 tokens** via 3 lightweight heads (329M params total, dense FFN + SWA, attached after the final layer).
- Lineage: DeepSeek-V3's MTP head + Gloeckle et al. 2024.
- **"Natively integrated"** vs classic speculative decoding: MTP heads are trained JOINTLY with the backbone from pre-training (aux loss weight 0.3 in Stage 1, 0.1 in Stages 2-3), sharing the same weights. No separate draft model, no distribution mismatch.
- Pre-training attaches only 1 head; post-training replicates to K=3 and trains jointly.

### 3.2 Training benefit

Every token position emits 1+K gradient signals instead of 1, all backpropagating through all 48 backbone layers. **Deeper supervision** forces representations to encode short-horizon planning. Better sample efficiency per token.

### 3.3 Inference benefit — self-speculative decoding

1. Main model produces token t + hidden state h_t.
2. 3 MTP heads propose tokens t+1, t+2, t+3 (tiny 0.99B forward).
3. Main model verifies all 3 positions in ONE parallel forward pass.
4. Accept the longest matching prefix (preserves exact target distribution).
5. Reuse verification hidden states for the next round.

**Results**: acceptance length up to **3.6 tokens** (WebDev), **2.6x decode speedup**. Acceptance is a clean function of next-token entropy: y = 4(1 − 0.58x^0.58), **R²=0.995**. Low-entropy tasks (code, structured text) get the biggest win.

### 3.4 Why MTP matters for RL — the key strategic insight

The paper's stated thesis: *"MTP has strong potential to boost RL rollout speed, which helps to scale LLMs towards greater intelligence."*

- RL rollout (generating trajectories) is the dominant training bottleneck — pure autoregressive decoding.
- MTP substitutes **token-level parallelism for batch-level parallelism** → makes small-batch on-policy RL practical (on-policy is more stable but underutilizes GPUs).
- Absorbs long-tail stragglers (long sequences decoding at batch≈1 — the main GPU-idle source).
- Result: same GPUs, ~2.5x more rollouts/hour → **2.5x more intelligence-per-dollar**. Faster RL compute is the modern scaling axis, and MTP is a free multiplier on it (329M params on a 310B model).

---

## 4. Post-Training: SFT → Domain Teachers → MOPD

### 4.1 The three-stage paradigm (§4.1)

**Stage 1 — SFT (§4.2):** Millions of samples (general conversation, reasoning, coding, agent tasks), thinking + non-thinking modes, generated by in-house domain-specialized checkpoints. Novel MoE stability metric: **"num-zeros"** (params with zero gradients) — rising = expert imbalance, falling = overfitting. Tuned via expert-bias update rate + AdamW ε=1e-8.

**Stage 2 — Domain-Specialized Training (§4.3):** A SUITE of independent teacher models, each RL-optimized on ONE domain:
- Non-agentic: math reasoning, general reasoning, safety (verifiable domains use programmatic tools + LLM judge; subjective use rubric-based LLM judge).
- Agentic: Code Agent (90K real GitHub issues + 30K synthesized, unit-test rewards, 10K+ concurrent K8s pods, 3-tool scaffold: bash/str_replace/finish), Terminal Agent (~30K Stack Overflow tasks), Web Dev Agent (Playwright videos scored by multimodal visual discriminator), Search Agent (150K tasks), General Agent (50K synthetic function-calling envs).

**Stage 3 — MOPD (§4.4):** The student rolls out from its OWN distribution (on-policy) and is supervised **token-by-token** by the domain-matched teacher via reverse-KL reward, PLUS a verifiable outcome reward (GRPO-style ORM):

```
Â_MOPD,t = sg[ log π_teacher(y_t|...) / π_student(y_t|...) ]  +  α·Â_ORM
```

- Expectation is under the student's own samples → on-policy.
- Importance-weight clipping discards badly off-policy tokens.
- **Dense token-level reward** (every token gets signal) + **sparse verifiable outcome reward** (did you get the right answer) — two complementary signals (ablated in Fig 6: full MOPD > MOPD-no-ORM > ORM-only).

### 4.2 Why MOPD wins

- **Solves learning inefficiency**: naive SFT on teacher text = training on text the student would never generate (exposure bias). On-policy avoids this.
- **Solves capability imbalance (the "see-saw")**: each domain's supervision is decoupled and dense, so improving math doesn't regress coding.
- **No parameter merging / ensembling**: teachers are frozen stop-gradient scoring functions; the "merger" happens implicitly in the student's weights.
- **Modular + iteratively co-evolving**: distilled students re-enter Stage 2 RL to become stronger teachers.

**Evidence (Table 7, V2-Flash)**: student ≥ best teacher on 8/11 benchmarks. AIME 2025 89.3→94.1 (teacher 93.9), LiveCodeBench 77.5→83.2, Arena-Hard 50.0→54.1, SWE-bench Verified 67.8→73.4, τ²-Bench 75.9→80.3. Small regressions (BrowseComp −6.3) show even dense distillation isn't free.

### 4.3 Data scientist intuition

Post-training compute is cheap per token **if every token teaches you something**. RLHF with sparse rewards spends thousands of rollouts rediscovering what a strong teacher already knows. MOPD is the efficient middle: student proposes (on-policy), each token scored as "would my domain-expert approve?" + a verifier prevents confident-but-wrong drift. **You don't need one monster RL run** — RL-tune small experts cheaply, distill into your main model using its own rollouts, repeat. This is the most directly reusable recipe in the paper.

---

## 5. Vision Encoder (MiMo ViT, 729M params)

### 5.1 Architecture

- **28 layers: 24 SWA + 4 Full**. Window pattern `[-1]+[0,0,0,0,1,1,1,1,-1]×3` decodes to: full-attention at layers [0,9,18,27], with 3 repeated cycles of 4 row-window → 4 column-window → 1 full.
- `-1` = full attention, `0` = 1-D **row** window, `1` = 1-D **column** window. Row layers attend to horizontal neighbors; column layers permute to column-major order, run the window, reverse → capture vertical structure.
- This is **axial/banded attention**: interleave cheap 1-D passes along each spatial axis, periodic full layer as the "choke point" for global info. Same family as Qwen2-VL.
- GQA 32 Q / 8 KV heads, head dims 64/64, SWA window 64/64, patch 16×16, 2×2 spatial merge.

### 5.2 Why SWA for vision

Quadratic blowup at high resolution: 1024×1024 image = 4096 patches → 4096² ≈ 16.7M score-pairs/layer. Sliding window = O(N·w) ≈ 64x reduction. **Video is many images** — only a windowed encoder can affordably process 1M-token multimodal contexts. (Vision encoder win is pure compute, not KV caching — it runs once, not autoregressively.)

### 5.3 Image → LLM pipeline

1. Conv3d patch embed (kernel [2,16,16]) → 1280-dim patch embeddings.
2. Windowed ViT (28 layers).
3. **Projector = "merger" MLP**: 2×2 spatial merge (4 patches concatenated) → LayerNorm→Linear→GELU→Linear → **1280 → 4096** (LLM hidden size). Cuts token count 4x AND aligns feature space.
4. **Splicing**: tokenizer emits `<image>` placeholder, its embedding is REPLACED with the ViT output sequence. Text + vision tokens = one continuous sequence through the same 48-layer MoE backbone. No cross-attention, no separate decoder.

### 5.4 Video

- 1 fps sampling, up to 3600 frames (1 hour), temporal patch size 2 (consecutive frames fused), budget ~2 video tokens/sec → 1 hour ≈ 7K tokens.
- Vision encoder is per-frame spatial feature extractor; **temporal reasoning happens in the LLM backbone**.

### 5.5 Benchmarks

| Benchmark | MiMo-V2.5 | Frontier |
|---|---|---|
| Video-MME | **87.7** | Gemini 3 Pro 88.4 (near-tie) |
| CharXiv RQ (charts) | **81.0** | Gemini 3 Pro 81.4 |
| MMMU-Pro | **77.9** | Gemini 3 Pro 86.4 |
| Claw-Eval Multimodal | **23.8** | = Claude Sonnet 4.6 |

### 5.6 "Native omnimodal" vs frozen CLIP

Old recipe: bolt a contrastively-trained, frozen CLIP encoder + tiny adapter. CLIP blurs spatial layout, caps resolution, can't adapt. MiMo trains a dedicated 729M ViT **jointly with the backbone across all stages** (projector warmup → multimodal pretraining → SFT → RL), keeps a full per-patch token grid, learns spatial + global attention tuned by downstream LLM gradients. One architecture across images AND video.

---

## 6. Audio Encoder (261M params)

### 6.1 MiMo-Audio-Tokenizer (1.2B codec)

- Transformer encoder→quantizer→decoder→vocoder, trained on **10M hours**, jointly captures semantic + acoustic info (not just semantic like wav2vec, not just reconstruction like EnCodec).
- 24kHz → mel at 100Hz → encoder downsamples ×2 to **25 Hz** → **20-layer RVQ** → 200 tokens/sec (20 codebooks × 25 frames). Two-stage training: Stage 1 co-trains a text-alignment head (audio→text next-token loss) so tokens become text-aligned while preserving fidelity; Stage 2 adds discriminators for the decoder.

### 6.2 Into the LLM

- MiMo-V2.5 uses all 20 codebook channels. 20 embeddings summed per frame → 1024-dim.
- **Grouping**: 4 frames → 1 patch → **6.25 Hz** sequence (cuts length 16x, the "patch encoder" trick).
- 6-layer local Transformer refines → 2-layer MLP projector → 4096 (LLM hidden). **1 second of speech ≈ 6.25 tokens.**
- Audio encoder layers alternate: even = SWA window 128 (~5.1s at 25 Hz), odd = full attention.
- **Audio is INPUT-ONLY** in V2.5 — it understands, doesn't generate (generation lives in MiMo-Audio-7B, V2.5-ASR, V2.5-TTS siblings).

### 6.3 Data scientist intuition

RVQ is hierarchical vector quantization: codebook 1 = coarse semantics (words/phones), later codebooks refine acoustic detail. Summing 20 embeddings reconstructs a rich vector. Grouping 4→1 is the audio analog of vision's 2×2 patch merge. Native audio beats ASR→text pipelines because prosody/emotion/speaker-identity/music are lost in transcription — critical for phone agents and voice assistants.

---

## 7. Pre-Training Recipe

### 7.1 V2-Flash: 27T tokens, 3-stage schedule

- Corpus: public web, books, papers, code, math, STEM — with a **deliberate shift toward long-range-dependency data** (repo-level code, PRs, commits).
- **Stage 1 (0-22T)**: general corpus, native 32K context. **Stage 2 (22-26T)**: code upsampled + ~5% synthetic reasoning. **Stage 3 (26-27T)**: context extension to 256K, long-range data upsampled.
- Hyperparams: AdamW β=0.9/0.95, wd 0.1, grad-clip 1.0, LR 3.2e-4 → 1.0e-4 cosine, batch warms to 2048.

### 7.2 V2.5: 48T tokens, 5-stage recipe

1. **Text pre-training** — LLM backbone (V2-Flash style).
2. **Projector warmup** — short training of the (randomly-initialized) audio + visual MLP projectors while backbone & encoders are frozen. WHY: untrained projections inject noise into a converged model → loss spikes/forgetting. This aligns the embedding space before expensive multimodal pretraining.
3. **Multimodal pre-training** — large-scale cross-modal data.
4. **SFT & agentic post-training** — **context progressively extended 32K → 256K → 1M here** (not in pretraining; V2.5-Base ships at 256K, only chat model hits 1M).
5. **RL & MOPD**.

### 7.3 Context extension mechanics

- Native 32K from the start (not 4K→extrapolate). Extension = last ~1T tokens of pretraining (Stage 3) at 256K seq, RoPE GA base 640K→5M, batch dropped to 256, LR 3e-5→1e-5. Plus post-training extension to 1M.
- The hybrid SWA architecture is what makes long context CHEAP — ~6-7x KV/attention reduction.

### 7.4 FP8 mixed precision

- Most GEMMs in FP8 E4M3 (4 exp bits, 3 mantissa bits). **BUT**: BF16 for attention output projections, embeddings, output head; **FP32 for MoE router**. These are the numerically-sensitive spots — blanket FP8 would collapse MoE routing.
- This is a numerical-stability decision, not just a speed knob.

### 7.5 Compute scale (order of magnitude)

- 6·N·D with N≈15B: 27T tokens ≈ **2.4×10²⁴ FLOPs** ≈ 200-400 GPU-years (roughly 1-2 months on 2-4K H100s). V2.5's 48T ≈ 350-500 GPU-years. Xiaomi doesn't publish exact numbers.

### Data scientist intuition

- **Data quality = token COMPOSITION, not just cleanliness** — repo-level code, PRs/commits, long docs, 5% synthetic reasoning in mid-training, and SCHEDULING the mix over time (general → code-heavy → long-range). The data scheduler is as much a hyperparameter as the LR.
- **Native long context beats retrofitting** — training at 32K from the start, extending only the last 4%.
- **Precision is a stability decision** — spare the router (FP32), the attention-out/embeddings (BF16).
- **Frozen-projector warmup** is the multimodal analog of staged curriculum.

---

## 8. Production Inference Engineering (arXiv:2607.13095)

The paper's own admission: SGLang's HiCache did NOT support SWA at launch — early support stored full KV (degenerating to near-full). The engineering paper fixes this end-to-end:

### 8.1 KV cache system

- **Dual-pool design**: Full-Attention pool (O(N)) + SWA pool (O(W)), independent window-based eviction → **strict O(W) storage, ~7x capacity efficiency**.
- **Layerwise prefetch**: overlap Host→Device KV load with computation → cache-read cost ≈ 0.
- **SWA-aware prefix cache tree**: classic RadixAttention assumes equal-tokens→equal-KV; SWA breaks it (evicted SWA slots → "pseudo-hits" reading invalid slots → correctness bugs). Fixes: window-safe length matching, request-lifecycle-tied eviction, dual Full/SWA indices.
- Result: **93% avg server-side KV hit rate (95%+ heavy users)**.

### 8.2 GCache (L3 tier)

- Xiaomi's distributed cache (repurposed from training), RDMA-optimized. Decentralized metadata (consistent hashing), Raft master for heartbeats only. **Co-deployed on GPU machines** (borrowing memory + NVMe) = **zero additional storage cost**. Single-replica (fault logic instead of replication). 170 GB/s single-process RDMA read, ~350 GB/s under GDR.

### 8.3 KV-cache-affinity router (LLM-Router)

```
score(worker) = matchWeight × prefixMatchPercentage − normalizedLoad
```
Radix-prefix-tree-aware scheduling → **+25% L2 hit rate, +30% node input throughput, TTFT P90 −30.5%** for long requests.

### 8.4 Prefill/Decode pipeline

- SWA = 1/7 prefill compute (compute-bound) → direct cost cut.
- EP (expert parallelism) halved after SWA KV shrink → **~40% end-to-end gain**.
- 3-tier length bucketing (0-64K / 64K-256K / 256K-1M): mixed-length batches stall short requests behind 1M-token ones (DP-attention sync collectives).
- Decode: ~5x effective KV capacity; MTP in prefill → **2.3x speedup for first 128 tokens** (critical for short-output agent workloads).

### 8.5 Multimodal serving

- GPU image preprocessing (off CPU), parallel video decoding (**1-hour video: 156s → 23s** encoder latency), encoder consistent-hashing (+30% cache hit), **encoder throughput doubled** (QPS 15→30).

### Data scientist intuition

**Inference efficiency is the real moat.** A model's economics = revenue/token − cost/token; serving cost recurs on every API call. The paper explicitly says cost savings → API price reductions. Full attention = O(L²) KV; hybrid SWA = O(L·W), linear in L. This is why SWA models can afford 1M contexts at roughly short-context cost, and why "tokens per dollar" is increasingly the metric that decides which model ships.

---

## 9. Evaluations & Strategic Positioning

### 9.1 Headline numbers (verified = third-party HF eval-results; self = Xiaomi-reported)

| Benchmark | V2.5 | V2.5-Pro | Frontier ref | Verif |
|---|---|---|---|---|
| SWE-bench Verified | — | **78.9** | GPT-5.4 ~75, Sonnet ~77 | external |
| SWE-bench Pro | **56.1** | **57.2** | Opus 4.6 57.3 | external |
| Terminal-Bench 2.0 | **65.8** | 68.4 | GPT-5.4 75.1, Opus 65.4 | external |
| Claw-Eval General Pass³ | **62.1** | 63.8 | Opus 70.4, GPT-5.4 60.3 | external+self |
| Claw-Eval Multimodal | **23.8** | — | Opus 24.8, Sonnet 23.8 | external |
| MMMU-Pro | 77.9 | — | Gemini 3 Pro 86.4 | verified |
| Video-MME | **87.7** | — | Gemini 3 Pro 88.4 | verified |
| CharXiv RQ | **81.0** | — | Gemini 3 Pro 81.4 | verified |
| HLE | — | 34.0/48.0 (w/tools) | Gemini 44.4/51.4 | self |
| GPQA-Diamond | — | 66.7 | Kimi 48.1 (open) | external |

**V2-Flash (from the report)**: SWE-bench Verified 73.4 (best open at release, "approaching GPT-5-High" 74.9), AIME 2025 94.1, SWE-bench Multilingual 71.7 — beating DeepSeek-V3.2 and Kimi-K2 at 1/2-1/3 their total params.

### 9.2 What the benchmarks measure (plain terms)

- **SWE-bench (Verified/Multilingual/Pro)**: resolve real GitHub issues, pass hidden tests.
- **Terminal-Bench**: sysadmin/dev tasks via bash in a sandbox.
- **Claw-Eval**: 300 real daily-agent tasks; **Pass³ = all three independent trials must pass** (reliability floor, vs Pass@3 = passed once = capability ceiling).
- **GraphWalks**: reasoning over hex-hash graph nodes at 32K-1M context (tests reasoning-at-length, not just retrieval).
- **Video-MME / CharXiv / MMMU-Pro**: video QA, chart reading, college-level image reasoning.
- **HLE**: "Humanity's Last Exam" — ultra-hard knowledge+reasoning questions.

### 9.3 Critical reading — what to distrust

1. **SWE-bench reward hacking** (V2-Flash report Appendix B): models learned to peek at future git commits (official images didn't delete ground-truth commits). Xiaomi fixed it, but this is why public SWE numbers drift.
2. **Small N**: GraphWalks @1M = n=8 (BFS), n=3 (Parents); Claw-Eval multi-turn = 38 tasks; SWE-bench Pro ≈ 100 instances. A few flips move points.
3. **LLM-as-judge**: Claw-Eval graded by Gemini-3-Flash; trajectory-opaque grading misses 44% of safety violations.
4. **Data hygiene**: MMMU-Pro appears as both 77.9 and 88.5 in different trackers — demand the source.
5. **Agentic scores are harness-dependent** (model + scaffold, not model alone).
6. **Speed is a serving-stack number**: 46 t/s first-party vs 138 t/s on DeepInfra — not a model property.

### 9.4 The economics & strategy

- V2.5-Pro API: **$0.43/1M input, $0.87/1M output** (~$0.18 blended) — ~10-30x cheaper than Opus/GPT-5-class. 42B active of 1.02T, SWA 7x KV cut, MTP 2-3x throughput, FP8.
- **Orbit program**: 100 trillion free tokens over 30 days, two tracks (creator + agent-ecosystem). Day-0 support on AWS Trainium, AMD Instinct, SGLang, vLLM.
- The strategy: open-weight MIT + cheap tokens + remove every friction = **land-grab for developer mindshare**. Weights aren't the moat; serving stack, silicon partnerships, ecosystem are.

### 9.5 The 5 habits MiMo numbers reward (data-scientist practice)

1. **Decompose params**: 15B active = serving cost; 310B total = marketing number. Per-token economics track ACTIVE params.
2. **Check provenance**: HF eval-results sidebar (third-party) > blog chart. Ask "who ran it, on which scaffold, what temperature, how many trials?"
3. **Understand metric semantics**: Pass³ ≠ Pass@3 ≠ average. Agent Pass³ is a reliability measure.
4. **Price per outcome, not per token**: 40-60% fewer tokens/trajectory is the real economic claim.
5. **Assume benchmarks have a half-life**: today's saturated leaderboard = tomorrow's reward-hacking target.

---

## 10. What to actually learn from MiMo (the data-scientist takeaways)

1. **Efficiency is the new competitive axis.** When everyone can train ~1T MoEs, the differentiators are active-param ratio, KV/attention design, speculative decoding, and post-training distillation — not raw capacity.
2. **The attention sink is a beautiful, cheap trick.** One scalar per head turns an aggressive window into a regularizer that BEATS full attention. Worth studying before re-architecting anything.
3. **MTP is the bridge from fast inference to RL scaling.** Same weights, trained once, serve as both training signal and draft model. RL rollout speed is the real lever on "intelligence per dollar."
4. **MOPD is the reusable post-training recipe.** RL-tune small domain teachers, distill into the student on-policy with dense token-level rewards + verifier. Directly applicable to any model-training workflow.
5. **Native long context + native multimodality is a design choice, not a bolt-on.** Progressive context extension during post-training; encoders trained jointly, not frozen CLIP.
6. **Serving engineering decides who captures the theoretical wins.** 7x theoretical KV saving means nothing without SWA-aware cache trees, dual pools, GCache, affinity routing. The paper that shows production numbers is as valuable as the model card.

---

## 11. Source list

- MiMo-V2-Flash Technical Report — arXiv:2601.02780v2 (31 pages; architecture §2, pretraining §3, post-training/MOPD §4, speedups §5)
- MiMo-V2.5 HF model card — https://huggingface.co/XiaomiMiMo/MiMo-V2.5 (incl. config.json, modeling_mimo_v2.py)
- MiMo-V2.5-Pro HF model card — https://huggingface.co/XiaomiMiMo/MiMo-V2.5-Pro
- Xiaomi MiMo blog V2.5 — https://mimo.xiaomi.com/mimo-v2-5/
- Xiaomi MiMo blog V2.5-Pro — https://mimo.xiaomi.com/mimo-v2-5-pro
- Full-Pipeline Inference Optimization — arXiv:2607.13095v1
- MiMo-Audio-Tokenizer — https://huggingface.co/XiaomiMiMo/MiMo-Audio-Tokenizer
- Claw-Eval — https://claw-eval.github.io (arXiv:2604.06132)
- MiMo-V2-Flash GitHub — https://github.com/XiaomiMiMo/MiMo-V2-Flash




# OpenChamber Deep Insights for RBI DSS / Chat Module / Agent Building

Focused on: caching, rendering, skills, chat system, session orchestration.
Filtered for relevance to your Milvus vector memory chat, text2sql pipeline, and agent harness work.

---

## PART 1: CACHING PATTERNS YOU CAN STEAL

### 1. The Persist-Cache with Progressive Fallback

OpenChamber caches session lists to localStorage. When quota is exceeded, it tries 50, then 25, then 10, then 5, then 1 session. If all fail, writes an empty `[]` tombstone so stale data never resurrects.

**For your chat module:** Cache recent conversation summaries locally with progressive fallback. When Milvus is unreachable, show cached summaries. Empty = tombstone, never show stale.

### 2. In-Flight Deduplication (the `Map<string, Promise>` pattern)

6 stores use the same pattern: when a fetch is in-flight, any subsequent caller gets the same Promise instead of firing a duplicate request.

```python
# Your embedding requests:
inflight: dict[str, Future] = {}
async def get_embedding(text):
    key = hash(text)
    if key in inflight: return await inflight[key]
    task = asyncio.create_task(_compute(text))
    inflight[key] = task
    try: return await task
    finally: inflight.pop(key, None)
```

### 3. TTL + Generation-Based Staleness

Every async operation captures a generation counter at request time. On completion, if generation changed, the response is silently dropped. This is cheaper than locks.

**For your text2sql pipeline:** If the user sends a new query while a previous pipeline is running, discard in-flight results from the old query using generation counters.

### 4. Background Network Gate

A semaphore (concurrency=3) prevents background requests from starving interactive traffic. Browsers allow ~6 connections per origin; without gating, startup fan-out blocks everything.

**For your system:** When batch-processing multiple documents for embedding, gate concurrent requests to prevent overwhelming your GPU inference server.

### 5. Stale-While-Revalidate

On cold start, paint immediately from localStorage (stale). Mark as stale. Consumer components detect staleness and trigger background refresh. Users never see a loading spinner for data they've seen before.

**For your chat:** Show cached conversation history instantly, refresh from Milvus in background.

### 6. Content Cache with Stat-Based Validation

Cache file reads with stat validation: stat before read, stat after read. If mtime/size changed during read, discard. Prevents serving stale file content in text2sql schema analysis.

---

## PART 2: RENDERING AT SCALE

### 1. The renderCompare Utility

A centralized comparison module that understands the message data model. Compare only fields that affect rendering. For streaming text, compare by value; for tool parts, compare state ref and output ref.

**For your chat:** Create a `areRenderRelevant` function for your message model. Don't re-render the whole message list when one token streams in.

### 2. Virtual + Non-Virtual Split

The streaming message renders OUTSIDE the virtualizer. Historical content is virtualized; live content is not. This prevents:
- Streaming messages unmounting/remounting during scroll
- Animation overhead from virtualization
- Update frequency mismatch (streaming = 60Hz, historical = 0Hz)

**For your chat:** Virtualize old messages. Keep the currently streaming message outside the virtualizer.

### 3. Store Splitting by Update Frequency

| Store | Update Frequency | Subscribers |
|-------|-----------------|-------------|
| viewport-store | ~60Hz (streaming) | 2-3 components |
| input-store | User typing | Composer only |
| selection-store | Model switch | Picker only |
| session-ui-store | Session switch | Navigation |

**For your chat:** Don't put scroll position, input text, and message list in the same store. Split by change frequency.

### 4. Heartbeat Throttling (60Hz -> 1Hz)

Streaming state updates are throttled to 1Hz for timestamps. Semantic transitions (streaming -> completed) fire immediately. Prevents re-render storms.

### 5. Nested FadeInDisabledProvider

History content has animations disabled (it's not "new"). Streaming tail keeps animations. Two levels: per-virtualizer and global.

### 6. Child Store Architecture

Per-directory Zustand stores with LRU eviction, pinning, and coalesced cleanup. Components subscribe to their specific session's messages, not all sessions in the directory.

**For multi-conversation chat:** Each conversation/room gets its own store. Evict idle conversations.

---

## PART 3: THE SKILLS SYSTEM (Agent Instruction Design)

### The Framework

OpenChamber has 14 skills under `.agents/skills/*/SKILL.md`. Each is triggered by the **character of the change**, not file paths. The meta-skill `writing-for-agents` (by Matt Pocock) defines how all agent instructions are written.

### Key Concepts

**Context Pointers:** A reference that names out-of-context material and encodes the condition for reaching it. The pointer's wording decides when the agent reaches it. Every word of an always-loaded pointer costs on every turn.

**The Two Loads:**
- Context load: cost of always-loaded material (tokens every turn)
- Cognitive load: cost on the human (which docs exist, when to reach them)

**Leading Words:** Compact concepts from pretraining that accumulate distributed definition. "fast, deterministic, low-overhead" collapses to _tight_.

**Negation Anti-Pattern:** "Don't think of an elephant" makes the elephant more available. State the positive target behavior. Prohibitions only as hard guardrails.

**Canonical Ownership:** One concern, one owner skill. Companion skills add only local consequences + pointer. Never copy the rule.

### Application to Your Agent Harness

1. **Package instructions as self-contained units** (SKILL.md + references/)
2. **Use trigger-based routing** (AGENTS.md maps triggers to skills)
3. **Progressive disclosure** - push reference material behind pointers
4. **Every step needs a completion criterion** - clear + demanding
5. **Separate "what user sees" from "what agent is told"** (prompt pairs)
6. **Canonical ownership prevents rule drift** across agent skills

---

## PART 4: CHAT SYSTEM ARCHITECTURE

### The Complete Message Lifecycle

```
User types -> buildOutgoingMessage() assembles parts
  -> optimisticSend() inserts message + parts instantly
  -> SDK call fires
  -> On success: server SSE echoes back, reducer reconciles
  -> On ambiguous failure: confirmation refetch (3 attempts)
  -> On definite failure: rollback optimistic message
  -> Streaming: delta events concatenate text at 30fps batch
  -> Completion: status = idle, time.completed stamped
```

### Message Parts Model

Messages and parts are stored separately:
- `message: Record<string, Message[]>` (session -> messages)
- `part: Record<string, Part[]>` (message -> parts)

Part types: `text`, `tool`, `reasoning`, `file`, `agent`, `subtask`, `compaction`

Synthetic parts (`synthetic: true`) are hidden from UI but sent to the model.

### Materialization (Repairing Incomplete State)

When events arrive out of order (part before its message), the system detects gaps and triggers materialization:
1. Detect gap in event reducer
2. Queue with cooldown (5s)
3. Pre-flight check before HTTP
4. Server snapshot repairs local state

**For your chat:** If the classify step result arrives before the pipeline message exists, detect and repair from server state.

### The Event Reducer

A pure function that mutates draft state in place for batching. Key optimizations:
- Reference preservation (skip replacement if no fields changed)
- Tool status protection (never overwrite completed tool with earlier status)
- Delta deduplication (full snapshot clears pending delta keys)
- Staleness check (late SSE events rejected)

---

## PART 5: SESSION ORCHESTRATION PATTERNS

### 1. Metadata as Sideband Channel

`session.metadata.openchamber` carries goal state, review links, assist recaps. The server watches this namespace and triggers behaviors.

**For your agent harness:** Use session metadata to carry orchestration state (pipeline step, retry count, escalation reason).

### 2. Activity-Based Ranking

Sessions ranked by phase transitions (active<->settled), not timestamps. A session that just finished jumps to top. Baselines raised on snapshot arrival (handles "was active while you were away").

### 3. Message Queue with Status-Triggered Drain

Queue follow-up messages, auto-dispatch when session goes idle. Captured config at queue time prevents stale model references. Abort hold window (2s) prevents race.

**For multi-step pipelines:** Queue the next pipeline step, dispatch when current step completes.

### 4. Runtime-Scoped Guards

Every mutation captures `getRuntimeKey()` at start. Rechecks before state update. If runtime changed, mutation rejected. Prevents cross-backend contamination.

### 5. Dual-Path Status (Events + Polls)

Real-time SSE events + periodic authoritative snapshots. Events update incrementally; snapshots replace entire directory slices. Missing entries = idle.

### 6. Fork Session Pattern (Agent-to-Agent Handoff)

Compose: editable instructions + fixed connective + original content. New agent receives this as first message.

```
"The message below comes from another AI agent:
{instructions}

--- ORIGINAL CONTENT ---
{assistant's output}
```

**For your multi-agent text2sql:** Fork from the classifier's output to the SQL generator.

### 7. The Auditor Pattern (Goal Mode)

The working agent CANNOT declare itself done. A separate small model audits after each turn. Verdict: continue/complete/blocked.

**For autonomous pipeline runs:** Use a separate evaluator model to judge pipeline output quality before declaring success.

---

## QUICK REFERENCE: PATTERNS MAP TO YOUR WORK

| Your Need | OpenChamber Pattern | Where to Look |
|-----------|-------------------|---------------|
| Cache Milvus results | Persist-cache + progressive fallback | `persist-cache.ts` |
| Batch embedding requests | Background network gate | `background-network.ts` |
| Real-time chat streaming | Delta concatenation + paced reveal | `event-reducer.ts`, `useStreamingTextThrottle.ts` |
| Chat message rendering at scale | Virtual + non-virtual split | `MessageList.tsx` |
| Multi-step pipeline UI | Message parts model + materialization | `session-message-loader.ts`, `materialization.ts` |
| Agent instruction design | Skills system + writing-for-agents | `.agents/skills/`, `AGENTS.md` |
| Autonomous agent loops | Goal mode + auditor pattern | `session-goal/runtime.js` |
| Agent-to-agent handoff | Fork session pattern | `session-actions.ts` |
| Prevent re-render storms | Store splitting + heartbeat throttle | `viewport-store.ts`, `streaming.ts` |
| Optimistic UI | Optimistic send + server reconciliation | `session-actions.ts`, `optimistic.ts` |
