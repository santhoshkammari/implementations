---

## 📊 Research Report: AI/ML Interview Patterns & Preparation Strategies for 2026

**Summary:** Based on research across 6+ authoritative GitHub repositories (khangich/machine-learning-interview, alirezadir/Machine-Learning-Interviews, alirezadir/Agentic-AI-Systems, shevchyk-serhii/scalas, julien731/obsidian-pkm, alexeygrigorev/ai-engineering-field-guide, JahnaviSingaraju/career-guidance-platform) plus primary industry sources, a complete picture emerges of a rapidly shifting interview landscape where LLM engineering, RAG, and agentic systems are now **the highest-signal topics**, traditional algorithm-only screens are being replaced by AI-enabled coding challenges, and companies are re-engineering their entire hiring rubrics around "builder" and "product engineer" archetypes.

---

## 1. Repositories Discovered

| Repository | Stars/Authority | Purpose |
|---|---|---|
| `khangich/machine-learning-interview` | 13K+ stars | Minimum viable MLE study plan, real FAANG interview questions |
| `alirezadir/Machine-Learning-Interviews` | High authority | Full ML interview guide — Google, Meta, Amazon, Apple, Roku offer recipient |
| `alirezadir/Agentic-AI-Systems` | Updated 2025 | Agentic AI system design — now officially Chapter 5 of ML interviews guide |
| `shevchyk-serhii/scalas` | 60KB guide | Comprehensive AI Engineer Interview Prep (USA 2025-2026) |
| `julien731/obsidian-pkm` | Primary research | AI-native hiring practices 2025-2026 across Sierra, Anthropic, Meta, Canva, etc. |
| `alexeygrigorev/ai-engineering-field-guide` | Curated list | Awesome ML/AI interview experiences + industry links |
| `JahnaviSingaraju/career-guidance-platform` | India-focused | Indian ML job market guide (Swiggy, Flipkart, Meesho, etc.) |

---

## 2. Interview Structure at Top Companies

### 🏢 Standard Process: 4–5 Rounds (2025-2026)

```
Round 1: Phone Screen (45-60 min)
  - Python coding + ML basics ("explain overfitting", "cross-validation")
  - Background ("walk me through your most impactful ML project")
  - Motivation ("why AI engineering?", "why this company?")

Round 2: Coding Round (60-90 min)
  - Python algorithms: trees, graphs, dynamic programming
  - Data manipulation: NumPy/Pandas problems
  - ML coding: implement gradient descent, k-means, attention from scratch

Round 3: ML System Design (60 min) ← MOST DIFFERENTIATING ROUND
  - Design end-to-end ML systems
  - Common: recommendation system, search ranking, content moderation, RAG pipeline
  - Evaluated on: requirements, data design, model selection, serving, monitoring

Round 4: ML Depth / Technical Deep Dive (60 min)
  - Transformer internals, RLHF, DPO, optimization
  - "Explain a recent paper you read"
  - Practical experience: "How did you handle data quality at scale?"

Round 5: Behavioral (45-60 min)
  - Cross-functional collaboration, ambiguity, project leadership
  - Failure stories, ethics/responsible AI
```

*Source: `shevchyk-serhii/scalas:job-prep/AI_ENGINEER_INTERVIEW_PREPARATION_USA.md:1-100`*

---

### 🔬 Company-Specific Variations

| Company | Interview Flavor | Key Differentiators |
|---|---|---|
| **OpenAI / Anthropic** | Research-heavy | LLM internals, safety/alignment, constitutional AI, RLHF deep dive. All roles internally titled "Member of Technical Staff" |
| **Google DeepMind** | Research-oriented | Paper discussions, deeper theory, strong math proofs |
| **Meta** | Coding + System Design | Strong LeetCode medium/hard focus + **AI-Enabled Coding pilot** (Oct 2025): multi-file codebase, choose your AI assistant (Llama/Claude/GPT) |
| **Amazon** | Applied Science track | ML case studies, coding (medium), behavioral (leadership principles heavy) |
| **Microsoft AI** | MTS pattern | Product focus, "AI Product Engineer" framing, AI fluency assumed |
| **Apple ML** | On-device ML | Privacy-first ML, efficient inference, Core ML, on-device models |
| **Tier 2 startups** | More practical | Less algorithmic, more "build it" mentality, take-home projects |

*Sources: `shevchyk-serhii/scalas:...USA.md` (company-specific section); `julien731/obsidian-pkm:resources/AI-native hiring practices 2025-2026.md:1-150`*

---

### 🇮🇳 Indian Startups (Flipkart, Swiggy, Meesho, etc.)

**Top AI/ML hiring companies in India:**
- **Product companies (highest pay):** Google India (DeepMind India), Microsoft, Amazon, Meta AI, Uber, Walmart
- **Indian product startups (growing fast):** Swiggy, Zomato, Flipkart, Razorpay, CRED, PhonePe, **Meesho**, Dunzo, **Sarvam AI**, **Krutrim**, Yellow.ai, Observe.AI
- **Analytics firms:** Mu Sigma, Fractal Analytics, LatentView, Tiger Analytics
- **Service companies (volume hiring):** TCS AI, Infosys AI, Wipro AI Labs

**Indian ML Interview Format (general pattern):**
1. **Coding Round:** DSA (LeetCode easy-medium), SQL queries — *"asked in almost every DS/ML interview"*
2. **ML Concepts Round:** Theory (algorithms, math, statistics)
3. **ML Case Study / Take-Home:** Build a model on given dataset
4. **System Design for ML:** Design recommendation system, fraud detection, search ranking
5. **Behavioral Round:** Project walkthroughs, challenges

**Salary ranges India (2024-2025):**
- ML Engineer fresher: ₹8-20 LPA
- Deep Learning Engineer fresher: ₹10-25 LPA
- NLP/LLM Engineer fresher: ₹10-22 LPA
- RAG building is the **"hot skill in 2024-2025"**

*Source: `JahnaviSingaraju/career-guidance-platform:career_docs/07_ai_ml_engineer_guide.txt`*

---

## 3. Round Types — Deep Dive

### 🧩 DSA/Coding Rounds

**LeetCode distribution (from actual 350-problem preparation data):**
- **55% Medium, 35% Easy, 15% Hard**
- LC Hard appears in **5-10% of actual interview questions**

**Key categories for MLE roles:**
- Arrays, strings, hashmaps (most common)
- Trees and graphs (BFS/DFS)
- Dynamic programming
- Sliding window, two pointers
- Heap/priority queue
- SQL (joins, window functions, CTEs)

**Important note:** *"There are a lot of companies that do NOT ask LeetCode questions. There are many paths to become an MLE."*

*Source: `alirezadir/Machine-Learning-Interviews:src/lc-coding.md`*; `khangich/machine-learning-interview:faqs.md`*

---

### 🏗️ ML System Design

**The 9-Step ML System Design Formula** (from alirezadir's guide, widely cited):

1. **Problem Formulation** — clarify, translate to ML objective
2. **Metrics** — offline (precision, NDCG, BLEU) + online (CTR, conversion rate)
3. **Architectural Components** — ML + non-ML modules
4. **Data Collection & Preparation** — labeling, weak supervision, transfer learning
5. **Feature Engineering** — embeddings, preprocessing, feature store
6. **Model Development** — KISS → iterate; LR → trees → neural nets → ensembles
7. **Prediction Service** — batch vs online vs hybrid, on-device AI, compression
8. **Online Testing & Deployment** — A/B experiments, canary, shadow mode
9. **Scaling, Monitoring, Updates** — drift detection, continual training

**Most common ML system design questions:**
- Recommendation systems (Netflix, YouTube, Spotify)
- Search and ranking (ads ranking, news feed, e-commerce)
- Content moderation (hate speech, spam, NSFW)
- RAG-based Q&A / Customer support chatbot
- Fraud detection
- Autocompletion / typeahead
- Harmful content detection (multi-modal)
- Self-driving perception/prediction
- Named entity linking / NLP tagging

*Source: `alirezadir/Machine-Learning-Interviews:src/MLSD/ml-system-design.md:1-200`*

---

### 📚 ML Theory/Fundamentals

**Top 20 ML Algorithms Asked:**
1. Linear Regression (least squares, MLE connection)
2. Logistic Regression (sigmoid, cross-entropy loss)
3. Support Vector Machines (kernel SVM)
4. Decision Trees (split criteria, pruning)
5. Random Forest (bagging, variance reduction)
6. Gradient Boosting / XGBoost / LightGBM
7. K-Means Clustering
8. K-Nearest Neighbors (KNN)
9. Naive Bayes (Bayesian inference)
10. PCA / Dimensionality Reduction
11. DBSCAN
12. AdaBoost
13. Gaussian Mixture Models
14. HMMs (Viterbi algorithm)
15. Feedforward Neural Networks
16. CNNs (ResNet, EfficientNet, ViT)
17. RNNs / LSTMs / GRUs
18. Transformers (full architecture)
19. VAEs / GANs / Diffusion Models
20. CLIP / Contrastive Learning

*Source: `alirezadir/Machine-Learning-Interviews:src/ml-fundamental.md:1-150`*; `shevchyk-serhii/scalas:...USA.md`*

---

### 🤖 Deep Learning Theory Questions (2026)

**Key topics:**
- **Backpropagation:** forward pass, chain rule, BPTT for RNNs
- **Vanishing/Exploding Gradients:** ReLU vs sigmoid, residual connections, gradient clipping
- **Batch Normalization:** training vs inference behavior (running mean/variance)
- **Dropout:** inverted dropout, use in LSTMs
- **Attention Mechanism:** scaled dot-product, multi-head attention (must implement from scratch)
- **Transformer Architecture:** encoder-only (BERT) vs decoder-only (GPT), positional encoding, RoPE
- **BERT vs GPT:** bidirectional vs causal, MLM vs next-token prediction
- **Optimization:** SGD, Adam, AdamW (preferred for transformers), warmup + cosine annealing
- **Weight Initialization:** Xavier/Glorot (sigmoid/tanh), Kaiming/He (ReLU)
- **Loss Functions:** MSE, Cross-Entropy, Hinge, Contrastive, Huber

*Source: `shevchyk-serhii/scalas:...USA.md:15000-25000` (self-attention and transformer implementation blocks)*

---

### 🗣️ NLP/LLM Questions Specific to 2026

**This is the "highest-signal topic" for 2025-2026 interviews:**

| Topic | What's Asked |
|---|---|
| **RAG Systems** | Chunk size tradeoffs, embedding models, dense vs sparse vs hybrid retrieval, reranking, evaluation (MRR, NDCG) |
| **Fine-Tuning** | LoRA vs QLoRA vs full fine-tuning, rank selection, target modules |
| **RLHF / DPO** | PPO + reward model vs direct preference optimization, when to use each |
| **Prompt Engineering** | Few-shot, chain-of-thought, structured output, system prompts |
| **LLM Evaluation** | ROUGE/BLEU, LLM-as-judge, task-specific evals, hallucination rate |
| **Context Window** | Handling limits, chunking, RAG vs long-context |
| **Hallucination** | Sources, mitigation strategies (RAG, guardrails, constitutional AI) |
| **Tokenization** | BPE vs WordPiece vs SentencePiece, why it matters |
| **Positional Encoding** | Sinusoidal, RoPE, ALiBi — why transformers need position info |
| **Agents** | ReAct pattern, tool use, multi-agent orchestration, MCP |
| **Safety** | Prompt injection, jailbreaking, data poisoning, output filtering |

**Key frameworks for 2025-2026:**
- LangChain / LangGraph, LlamaIndex, DSPy, Instructor, vLLM, Ollama

*Source: `shevchyk-serhii/scalas:...USA.md:25000-37000` (LLM Engineering section)*

---

### 👁️ Computer Vision Questions

- How does convolution work? What do learned filters represent?
- ResNet skip connections — why do they solve vanishing gradients in deep networks?
- CNN vs ViT — when to use each (CNNs = less data, ViTs = large-scale pre-training)
- Transfer learning — why effective, fine-tuning strategies
- CLIP — contrastive learning, zero-shot classification, multimodal embeddings
- Object detection: YOLO (one-stage) vs Faster R-CNN (two-stage), DETR (transformer)
- Segmentation: U-Net, Mask R-CNN, SAM (Segment Anything)
- Implement a CNN in PyTorch from scratch

*Source: `shevchyk-serhii/scalas:...USA.md:37000-47000` (CV section)*

---

### 📊 Statistics & Probability

- Bayesian inference, MAP vs MLE
- Bias-variance tradeoff (derive mathematically: Error = Bias² + Variance + Noise)
- A/B testing: null hypothesis, p-values, confidence intervals, statistical power
- Distributions: Gaussian, Binomial, Poisson, Bernoulli
- Probability puzzles (classic Bayesian: "2 kids, at least one girl..." type)
- Cross-entropy as MLE (relationship between maximum likelihood and loss function)
- KL Divergence: asymmetry, use in VAEs and RLHF KL penalty
- Information entropy: H(X), mutual information
- **Nobody cares about Gamma and Poisson distributions** ← direct quote from interview veteran

*Source: `khangich/machine-learning-interview:README.md` (statistics section); `alirezadir/Machine-Learning-Interviews:src/ml-fundamental.md`*

---

### 🧪 A/B Testing & Experimentation

- Trustworthy Online Controlled Experiments book (standard reference)
- How to set up A/B tests: control/test groups, null hypothesis, portion of users
- Statistical significance: p-values, confidence intervals, sample size
- Novelty effects, network effects, interaction effects
- Bandit algorithms vs traditional A/B (explore-exploit tradeoff)
- Shadow deployment, canary releases, rollback strategies
- Multi-armed bandits for personalization

*Source: `alirezadir/Machine-Learning-Interviews:src/MLSD/ml-system-design.md` (Step 8: Online Testing)*

---

### 🗃️ SQL in ML Interviews

SQL is asked in **"almost every DS/ML interview"**, especially at Indian product companies:

- JOINs: self join, inner, left, right, full outer
- Window functions: RANK(), ROW_NUMBER(), LAG/LEAD, rolling aggregates
- CTEs (Common Table Expressions)
- GroupBy + aggregation (SUM, COUNT, AVG per group)
- Subqueries and correlated subqueries
- Pandas equivalents (merge, groupby, pivot_table, rolling)
- Practice resources: HackerRank SQL, windowfunctions.com

*Source: `khangich/machine-learning-interview:README.md` (SQL section)*

---

## 4. Preparation Resources

### 📖 Essential Books (2026)

| Book | Use Case |
|---|---|
| **"Designing Machine Learning Systems"** by Chip Huyen | Production ML, system design — *essential* |
| **"AI Engineering"** by Chip Huyen (2025) | Building LLM applications — *new, highly relevant* |
| **"Hands-On Large Language Models"** by Alammar & Grootendorst | Practical LLM guide |
| **"Deep Learning"** by Goodfellow, Bengio, Courville | Comprehensive DL reference |
| **"Pattern Recognition and ML"** by Bishop | Mathematical ML rigor |
| **"Speech and Language Processing"** by Jurafsky & Martin | NLP fundamentals |
| **"Ace the Data Science Interview"** | Interview-specific, widely used |
| **"ML System Design Interview"** by Khang Pham | System design prep, #1 ML course on Educative |

---

### 🎓 Courses

| Course | Focus |
|---|---|
| Stanford CS229 (Andrew Ng) | ML fundamentals |
| Stanford CS231n | CNNs, Computer Vision |
| Stanford CS224n | NLP with Deep Learning |
| Stanford CS25 | Transformers United |
| fast.ai — Practical Deep Learning | Hands-on, production-oriented |
| DeepLearning.AI specializations | LLMs, MLOps |
| **Andrej Karpathy YouTube** | "Let's build GPT", neural nets from scratch — *highest recommended* |
| Educative.io: Grokking Coding Interview | LeetCode patterns systematically |
| Educative.io: Grokking ML System Design | ML system design |

---

### ⏰ Preparation Timelines

| Plan | Duration | For Whom |
|---|---|---|
| **Intensive** | 4 weeks | Experienced ML practitioners switching to AI Engineering |
| **Standard** | 8 weeks | Engineers with some ML experience |
| **Comprehensive** | 12 weeks | Career changers, building ML from scratch |
| **General estimate** | 3-4 months | *"3-4 months is a stretch but doable"* |

**4-Week Intensive Breakdown:**
- **Week 1:** Python/NumPy/Pandas + ML core fundamentals
- **Week 2:** Deep Learning (PyTorch loop + backprop + attention implementation)
- **Week 3:** LLM Engineering (RAG end-to-end) + Vector databases
- **Week 4:** System Design practice + MLOps + 2 full mock interviews

*Source: `shevchyk-serhii/scalas:...USA.md:47000+` (Study Plans section)*

---

## 5. Current Trends in AI Hiring (2025-2026) — Critical Findings

### 🔥 The Biggest Shift: Interview Redesigns

**Data point: In-person interview rounds rose from 24% (2022) → 38% (2025)** — driven by AI-cheating concerns.

**What's Being REPLACED:**
- Traditional algorithm/CS-fundamentals coding screens (Sierra, Canva explicitly replaced these)
- Spec-writing PM language
- Years of experience as gating criteria

**What's REPLACING them:**
- Multi-file codebase comprehension challenges (Meta's AI-Enabled Coding pilot — October 2025)
- Real project "Plan → Build → Review" sessions (Sierra's model — the industry reference)
- Paid 2-5 day work trials on real codebases (Linear, PostHog, 37signals)
- Live AI-assisted coding with screen share (OpenAI, Meta, Canva, Microsoft)

*Source: `julien731/obsidian-pkm:resources/AI-native hiring practices 2025-2026.md:1-200`*

---

### 🤖 Are Companies Asking About LLMs/RAG?

**Yes — it is now the PRIMARY differentiator:**
> *"This is the **highest-signal topic** for AI Engineer interviews in 2025-2026."* — shevchyk-serhii/scalas AI Engineer Guide

- DataCamp published **"Top 30 RAG Interview Questions for 2026"**
- DataCamp published **"Top 30 Agentic AI Interview Questions for 2026"**
- KDnuggets: **"10 Essential Agentic AI Interview Questions for AI Engineers"**
- TechEon: **"Complete Agentic AI System Design Interview Guide 2026"** (covering agent architecture patterns, autonomy boundaries, memory systems, tool use, evaluation)
- Top 50 LLM Interview Questions (Maxime Labonne, LinkedIn, highly circulated)

**RAG questions include:**
- Chunk size, overlap tradeoffs
- Dense vs sparse vs hybrid retrieval
- Reranking (Cohere, cross-encoders)
- Evaluation: Recall@k, MRR, NDCG, hallucination rate
- "Design a RAG system for customer support chatbot" (most common system design problem)

*Source: `alexeygrigorev/ai-engineering-field-guide:awesome.md` (AI Interview Experiences section)*

---

### 🕹️ Agent-Based System Design

**Agents are now Chapter 5 of the ML Interviews guide (2025 update):**
> `alirezadir/Agentic-AI-Systems` was created specifically because of hiring demand

**Agent interview topics:**
- ReAct pattern (Reason + Act loops)
- Tool use / function calling
- Multi-agent orchestration (LangGraph, CrewAI)
- Memory systems (short-term, long-term, episodic)
- Agentic RAG architectures
- Evaluation of agent behavior
- MCP (Model Context Protocol) — now a standard protocol in 2025
- Design patterns: reflection, planning, critique agents

**Augment Code's 6-dimension hiring rubric (most portable framework):**
1. Product & Outcome Taste
2. System & Architectural Judgment
3. **Agent Leverage** — *"Can you turn AI into real engineering throughput?"*
4. Communication & Collaboration
5. Ownership & Leadership
6. Learning Velocity & Experimental Mindset

> *"The human role is shifting from author to architect and editor."* — Augment Code

*Source: `julien731/obsidian-pkm:resources/AI-native hiring practices 2025-2026.md:200-350`*

---

### 💻 Practical Coding Tests (Build an ML Pipeline)

**Sierra's AI-native interview model (industry reference):**
1. **Plan:** Candidate drives ideation in their domain (2 hours solo, own AI tooling)
2. **Build:** Full codebase, candidate uses AI tools of their choice
3. **Review:** Demo + discussion of product decisions, code quality, data models, production readiness, and AI usage

**Canva's AI-Assisted Coding competency (June 2025):**
- Example problem: *"Build a control system for managing aircraft takeoffs and landings at a busy airport"*
- Candidates told to use AI (Cursor, Copilot, Claude recommended)
- Evaluated on: WHEN to use AI, breaking down ambiguity, reviewing AI-generated code, debugging AI failures

**Meta's AI-Enabled Coding pilot (October 2025):**
- Replaces ONE of two coding rounds
- 60 minutes, three-panel CoderPad: file explorer / editor / AI assistant
- Multi-file project with code they didn't write
- Pick from: Llama, ChatGPT, Claude, or Gemini

**Key insight across ALL redesigns:**
> *"The most successful candidates don't just prompt AI and accept whatever it generated. Instead, they asked thoughtful clarifying questions, used AI strategically for well-defined subtasks while maintaining control of the overall solution, and critically reviewed and improved AI-generated code."*

*Source: `julien731/obsidian-pkm:resources/AI-native hiring practices 2025-2026.md:300-450`*

---

### 📁 Portfolio/GitHub Importance

**GitHub portfolios matter significantly, especially in AI-native companies:**

> *"GitHub portfolios with deployed MVPs. Notebooks aren't enough. Hiring guides emphasise 'deployed MVPs, clear evaluation, and production patterns like hybrid retrieval, CI-gated tests, and observability.'"*

**Anti-signals:**
- Notebooks without deployment
- Demos without production evals or observability
- No clear evaluation methodology

**What works:**
- 8-10 repositories with clean READMEs
- 2-3 end-to-end projects (data collection → deployment)
- At least one deployed RAG or agent project
- Regular commits showing continuous learning
- Kaggle: Notebook Expert / Competitions Expert rank
- Technical blog posts (Medium, Towards Data Science, personal site)

*Source: `JahnaviSingaraju/career-guidance-platform:career_docs/07_ai_ml_engineer_guide.txt`; `julien731/obsidian-pkm:...2025-2026.md`*

---

## 6. Role Titles Emerging in 2025-2026

| Title | Companies | What it means |
|---|---|---|
| **Member of Technical Staff (MTS)** | OpenAI, Anthropic, Microsoft AI, Fireworks AI, Salesforce AI | Flat broad-band title. De-emphasises hierarchy. |
| **Product Engineer** | Linear, Cursor, PostHog, Vercel, Anthropic | Full-stack, blurs PM/eng, owns features end-to-end |
| **AI Product Engineer** | PostHog, Notion, Microsoft AI | Specialized in AI features: evals, prompting, agentic UX |
| **Forward Deployed Engineer (FDE)** | Anthropic, Palantir, OpenAI, Salesforce | Embedded with strategic customers, builds LLM apps in customer systems. **800% YoY spike in postings** |
| **Design Engineer** | Anthropic, Vercel, Linear, Cursor | Strong design sensibility, ships UI/UX directly |
| **Agent Builder** | Decagon, Walmart, SoFi | New — explicitly collapses PM/eng/design |

*Source: `julien731/obsidian-pkm:resources/AI-native hiring practices 2025-2026.md:Role Title Inventory section`*

---

## 7. MLOps/Deployment Questions (Production ML)

- How do you detect model/data drift in production? (PSI, KS test)
- Design a model serving system handling 10K QPS
- Training-serving skew — what is it, how to prevent
- Canary deployment vs A/B testing for ML models
- Feature stores (Feast, Tecton) — consistency between train/serve
- Monitoring: P50/P95/P99 latency, performance degradation
- CI/CD for ML: DVC, MLflow, GitHub Actions
- Model compression: quantization, pruning, knowledge distillation, LoRA
- Edge deployment: memory/compute constraints, ONNX, TorchScript

---

## 8. Key Code Implementations Interviewers Expect

Based on actual interview reports, these must be implementable from scratch:

```python
# 1. Softmax (numerical stability)
def softmax(x):
    exp_x = np.exp(x - np.max(x))
    return exp_x / exp_x.sum()

# 2. Cosine similarity
def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

# 3. Linear regression with gradient descent
# 4. K-means from scratch
# 5. Self-attention mechanism (full implementation expected)
# 6. PyTorch training loop (full: train + val phases, scheduler, gradient clipping)
# 7. Logistic regression from scratch
# 8. Backpropagation step-by-step
# 9. BERT vs GPT attention masks (causal vs bidirectional)
# 10. RAG pipeline (chunking → embedding → retrieval → generation)
```

*Source: `shevchyk-serhii/scalas:...USA.md:1-61KB` (code sections throughout)*

---

## 9. Gaps and Uncertainties

- **Indian startup-specific interview details:** General patterns confirmed for India, but company-by-company breakdowns (Meesho vs Swiggy vs Flipkart specifically) were not found in any indexed GitHub repos. Glassdoor/BlindApp would be better sources for this.
- **Apple ML specifically:** Only general tier placement found; Apple's interview is known to be secretive and hard to source.
- **Microsoft ML Engineer vs Azure AI Engineer:** Both exist but distinction in interview format not clearly documented.
- **OpenAI/Anthropic specific interview format:** Most data is from 2023-2024; processes may have changed with rapid org growth. The safety/alignment rounds are confirmed but not detailed.
- **Salary data for India:** Figures are from late 2024; may have shifted with the rapid growth of Sarvam AI, Krutrim, and similar Indian AI startups.
- **Key search limitation:** GitHub search did not surface India-specific detailed interview reports for Meesho, PhonePe, CRED; LinkedIn posts and Glassdoor would be better supplemental sources.

---

## Citations Summary

1. `khangich/machine-learning-interview:README.md` — Study guide, LC distribution, SQL, statistics, ML fundamentals
2. `khangich/machine-learning-interview:faqs.md` — Interview process FAQs, preparation timeline 3-4 months, LC hard 5-10%
3. `alirezadir/Machine-Learning-Interviews:src/MLSD/ml-system-design.md:1-450` — 9-step design formula, 30+ system design questions
4. `alirezadir/Machine-Learning-Interviews:src/ml-fundamental.md:1-200` — Top ML algorithms, DL topics, sample questions
5. `alirezadir/Machine-Learning-Interviews:src/lc-coding.md` — Coding round structure, 350 problems, 55% medium
6. `shevchyk-serhii/scalas:job-prep/AI_ENGINEER_INTERVIEW_PREPARATION_USA.md:1-61KB` — Complete 2025-2026 AI Engineer guide (company tiers, salary, code examples, study plans)
7. `julien731/obsidian-pkm:resources/AI-native hiring practices 2025-2026.md:1-350` — Industry research on hiring redesigns (Sierra, Meta, Canva, Augment Code, Anthropic) — created 2026-04-28
8. `JahnaviSingaraju/career-guidance-platform:career_docs/07_ai_ml_engineer_guide.txt` — Indian ML job market, company names, salary ranges
9. `alirezadir/Agentic-AI-Systems:README.md` — Agents as Chapter 5, agentic design patterns, frameworks
10. `alexeygrigorev/ai-engineering-field-guide:awesome.md` — AI interview experiences, RAG/agentic interview question resources for 2026