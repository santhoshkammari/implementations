---

## Summary

This research covers the full landscape of ML System Design for 2026 AI/ML engineering interviews — spanning classic ML systems (recommendation, fraud, search) and the rapidly growing GenAI/LLM/Agent category. The primary authoritative source discovered is `alexeygrigorev/ai-engineering-field-guide` (data-driven, 2,445 real job descriptions + 100+ interview reports), supplemented by `shafaypro/CrackingMachineLearningInterview` (complete code examples), `kengio/databricks-certification-study-guide` (production Databricks patterns), and `BhaveetKumar/nail-it` (backend+ML integration guide). Multiple web resources (Yuan Meng MLE Interviews 2.0, SystemDesignHandbook, InterviewNode, Chip Huyen GenAI Platform blog) provided practitioner perspectives.

---

## FULL RESEARCH FINDINGS

> **Source map:**
> - `alexeygrigorev/ai-engineering-field-guide:interview/questions/04-ai-system-design.md`
> - `alexeygrigorev/ai-engineering-field-guide:interview/questions/01-theory.md`
> - `alexeygrigorev/ai-engineering-field-guide:interview/05-trends.md`
> - `alexeygrigorev/ai-engineering-field-guide:learning-paths/README.md`
> - `shafaypro/CrackingMachineLearningInterview:system_design/recommendation_system.md`
> - `shafaypro/CrackingMachineLearningInterview:system_design/fraud_detection.md`
> - `shafaypro/CrackingMachineLearningInterview:system_design/ml_system_design_patterns.md`
> - `shafaypro/CrackingMachineLearningInterview:system_design/README.md`
> - `kengio/databricks-certification-study-guide:shared/interview-prep/14-ml-system-design.md`
> - `alexeygrigorev/ai-engineering-field-guide:role/04-use-cases.md`
> - Web: Yuan Meng `yuan-meng.com/posts/mle_interviews_2.0/`
> - Web: Chip Huyen `huyenchip.com/2024/07/25/genai-platform.html`
> - Web: systemdesignhandbook.com generative AI guide
> - Web: interviewnode.com GenAI system design patterns

---

```markdown
# ML System Design — 2026 Interview & Real-World Guide

**Last Updated:** 2026  
**Sources:** alexeygrigorev/ai-engineering-field-guide (2,445 real job descriptions), shafaypro/CrackingMachineLearningInterview, kengio/databricks-certification-study-guide, Yuan Meng MLE Interviews 2.0, Chip Huyen GenAI Platform blog, SystemDesignHandbook, InterviewNode

---

## Table of Contents

1. [The 2026 ML System Design Landscape](#1-the-2026-ml-system-design-landscape)
2. [Interview Format & What's Being Tested](#2-interview-format--whats-being-tested)
3. [The Universal Answer Framework](#3-the-universal-answer-framework)
4. [Classic ML System Design Topics](#4-classic-ml-system-design-topics)
   - [Recommendation Systems](#41-recommendation-systems)
   - [Fraud Detection](#42-fraud-detection)
   - [Search Ranking](#43-search-ranking)
   - [Ad Click Prediction](#44-ad-click-prediction)
   - [Content Moderation at Scale](#45-content-moderation-at-scale)
   - [Spam Classification](#46-spam-classifier)
   - [Anomaly Detection](#47-anomaly-detection)
   - [Feature Store Design](#48-feature-store-design)
   - [ML Pipeline Architecture](#49-ml-pipeline-architecture)
   - [A/B Testing Infrastructure](#410-ab-testing-infrastructure)
5. [GenAI/LLM System Design Topics (2026-Specific)](#5-genaillm-system-design-topics-2026-specific)
   - [RAG System Design](#51-rag-system-design)
   - [ChatGPT-Like System Design](#52-chatgpt-like-system-design)
   - [Agent System Design](#53-agent-system-design)
   - [Real-Time ML Serving at Scale](#54-real-time-ml-serving-at-scale)
   - [Fine-Tuning Infrastructure Design](#55-fine-tuning-infrastructure-design)
   - [Vector Search Systems at Scale](#56-vector-search-systems-at-scale)
   - [LLM Cost & Latency Optimization](#57-llm-cost--latency-optimization)
   - [LLM Safety & Guardrails](#58-llm-safety--guardrails)
6. [Key Concepts Deep Dives](#6-key-concepts-deep-dives)
   - [Online vs Offline Features](#61-online-vs-offline-features)
   - [Model Serving Patterns](#62-model-serving-patterns)
   - [Data Pipeline Architecture](#63-data-pipeline-architecture)
   - [Monitoring & Alerting](#64-monitoring--alerting)
   - [Model Retraining Strategies](#65-model-retraining-strategies)
   - [Scalability Considerations](#66-scalability-considerations)
   - [Cost Optimization](#67-cost-optimization)
   - [Latency Requirements](#68-latency-requirements)
7. [Common Interview Questions (2026)](#7-common-interview-questions-2026)
8. [Common Mistakes to Avoid](#8-common-mistakes-to-avoid)
9. [Books, Courses & Resources](#9-books-courses--resources)
10. [2026 Trends & Market Data](#10-2026-trends--market-data)

---

## 1. The 2026 ML System Design Landscape

### The Fundamental Shift

The center of gravity in ML system design interviews **has shifted dramatically**:

> "When models like GPT-4 and Claude became accessible via APIs, the hard part stopped being model training, and started being **system orchestration**." — InterviewNode

| Dimension | Classic ML System Design | GenAI / LLM System Design |
|-----------|--------------------------|---------------------------|
| **Focus** | Training pipelines, feature engineering | Orchestrating pre-trained models |
| **Data** | Feature engineering, training sets | Context engineering, chunking, retrieval quality |
| **Output** | Structured: scores, classifications | Open-ended: text, code, images |
| **Determinism** | Generally deterministic | Non-deterministic by default |
| **Evaluation** | Precision/Recall/F1/AUC on holdout | LLM-as-judge, human eval, task-specific evals |
| **Cost model** | Training compute (periodic) + serving | Per-token inference (continuous) + retrieval |
| **Failure modes** | Data drift, training-serving skew | Hallucination, prompt injection, context overflow, cost blowup |
| **Iteration speed** | Slow (retrain model) | Fast (change prompt, adjust retrieval) |

### What 2026 Interviewers Expect

Based on 130+ interview reports (Yuan Meng, Janvi Kalra 46 companies, Deepthi Sudharsan 50+ rounds):

1. **Classic ML** is still tested — especially at ranking-focused companies (Google, Meta, Netflix, LinkedIn, Pinterest, Snap)
2. **GenAI system design** is now a **separate, primary interview category** at AI-first companies
3. **AI/LLM questions tripled since 2023** across all ML roles
4. **Hybrid designs** are the new standard — systems that combine batch ML + LLM orchestration + agent loops
5. **Production thinking** is paramount — not just "how to build" but "how to evaluate, monitor, and scale"

### Companies Testing What

- **Ranking-focused** (Meta, LinkedIn, Netflix, Pinterest, Snap): Classic ML — recommendation, search, feeds, ads
- **AI-first** (OpenAI, Anthropic, Google DeepMind, Cohere, Salesforce): GenAI system design — RAG, agents, inference at scale
- **Enterprise AI** (Doctolib, Sprinter Health): Agentic system design — multi-step workflows, healthcare/domain-specific
- **AI Infra** (Modal, Fireworks, Pinecone): Model serving design, distributed GPU systems, vector search

---

## 2. Interview Format & What's Being Tested

### Standard Format (45-60 minutes)

You drive the conversation:

```
~5 min   → Clarify requirements, constraints, success criteria
10-15 min → High-level architecture sketch
15-20 min → Deep dive into key components
5-10 min  → Trade-offs, failure modes, bottlenecks
Remaining → Follow-up scenarios
```

*Source: alexeygrigorev/ai-engineering-field-guide:interview/questions/04-ai-system-design.md*

### What Interviewers Evaluate (5 Dimensions)

1. **Architectural reasoning** — Can you explain how components connect and interact (data → model → orchestration → feedback → governance)?
2. **Decision-making under constraints** — Can you balance creativity with control, cost with quality, latency with accuracy?
3. **Communication clarity** — Can you explain complexity without overwhelming the listener?
4. **Trade-off fluency** — retrieval speed vs context length, fine-tuning vs prompting, GPU cost vs latency
5. **Production thinking** — What breaks at scale? How do you monitor, alert, and recover?

### Seniority Expectations

| Level | What's Expected |
|-------|----------------|
| **Junior** | Focus on the prompt alone; high-level architecture |
| **Mid-level** | Describe embeddings or RAG mechanisms; basic system |
| **Senior** | Design an evolving ecosystem — document chunking strategies, how retrieval affects context windows, how outputs are validated and stored, how user feedback improves retrieval scoring |

> "At senior levels, system design interviews aren't about frameworks, they're about foresight." — InterviewNode

### Common Delivery Formats

- Whiteboard or virtual drawing (Excalidraw, Miro) — sketch components and data flows
- Discussion-based — walk through architecture verbally while interviewer probes specific areas

---

## 3. The Universal Answer Framework

### 6-Step Framework for Every ML System Design Question

```
STEP 1: CLARIFY REQUIREMENTS (5 min)
├── Functional: What must the system do?
├── Non-functional: Scale, latency, accuracy, cost, reliability
├── Business goal and success metric
├── Scale: users/day, QPS, data volume
├── Latency: acceptable p99?
├── Real-time or batch?
├── Data available?
└── Regulatory/compliance requirements?

STEP 2: DEFINE THE ML PROBLEM (5 min)
├── Convert business goal → ML objective
├── Input/output specification
├── Problem type: classification, regression, ranking, generation
├── Training signal (labels)
└── Offline vs online evaluation metrics

STEP 3: DATA DESIGN (10 min)
├── Data sources and collection
├── Feature engineering (online vs offline features)
├── Training/validation/test splits (temporal for time-series!)
├── Handling class imbalance, missing data
└── Data pipeline: ingestion → transformation → feature store

STEP 4: MODEL DESIGN (5 min)
├── Start simple (baseline), then complex
├── Justify model choice: data size, latency, interpretability
├── Architecture: classic ML vs LLM vs hybrid
└── Training approach and evaluation strategy

STEP 5: SERVING ARCHITECTURE (10 min)
├── Real-time (API) vs batch inference
├── Model serving infrastructure
├── Latency optimization: caching, model compression, quantization
├── A/B testing and canary deployment
└── Feature store for online serving

STEP 6: MONITORING & OPERATIONS (5 min)
├── What metrics to monitor? (data drift, prediction drift, business KPIs)
├── When to retrain? (triggers)
├── How to handle concept drift and data drift?
└── Rollback strategy
```

*Source: shafaypro/CrackingMachineLearningInterview:system_design/ml_system_design_patterns.md*

### For GenAI/LLM Questions — 5-Step Progression

```
1. Problem Framing (5-10 min)
   → clarify users, constraints, quality expectations, guardrails

2. High-Level Architecture (10-15 min)
   → core components and data paths from prompt to response

3. Deep Dive (20-30 min)
   → RAG design, tool use, memory, evaluation, safety

4. Trade-Offs (10-15 min)
   → what breaks, how you detect it, graceful degradation

5. Conclusion
   → summarize, list risks, outline next iteration
```

*Source: alexeygrigorev/ai-engineering-field-guide:interview/questions/04-ai-system-design.md*

### Four Repeatable Patterns (GenAI)

Most GenAI system design questions map to these patterns:

| Pattern | Use When |
|---------|----------|
| **RAG** | System orchestration and grounding accuracy (most common) |
| **Feedback & Reinforcement** | Implicit/explicit signals, active learning loops |
| **Hallucination Mitigation** | Retrieval-grounded pipelines, confidence estimation, source transparency |
| **Scalability & Cost Optimization** | Multi-layer caching, model tiering, prompt compression |

---

## 4. Classic ML System Design Topics

### 4.1 Recommendation Systems

**Reference designs:** Netflix, YouTube, Spotify, Amazon, TikTok  
*Source: shafaypro/CrackingMachineLearningInterview:system_design/recommendation_system.md*

#### Architecture: 3-Stage Pipeline

```
10M+ items in catalog
       ↓
STAGE 1: CANDIDATE GENERATION (< 20ms)
├── Two-Tower ANN Search (FAISS/ScaNN): top 200 items by embedding similarity
├── Trending Items: top 200 currently trending in user's region
├── Similar to Last Watched: top 200 items
└── Editorial/Promoted: curated items

Merge & deduplicate: ~1000 candidates

STAGE 2: RANKING (< 50ms)
├── Model: Wide & Deep / Gradient Boosting / Two-Tower with interaction features
├── Features: Rich user × item cross features, real-time context
└── Output: Predicted engagement probability (watch rate, completion rate)

STAGE 3: RE-RANKING
├── Diversity: Max 2 items from same genre in first 10 results
├── Freshness: Boost new releases < 30 days old
├── Promotions: Boost sponsored content (with disclosure)
├── Avoid recently watched (last 7 days)
└── Respect user preferences (disliked genres, blocked creators)

Final: Top 20-50 recommendations
```

#### Two-Tower Model (Core Architecture)

```
User Tower                    Item Tower
─────────────                ─────────────
user_id embedding            item_id embedding
age group                    genre embedding
country                      director embedding
watch_history_sequence       avg_rating
time_of_day                  release_year
device_type                  duration
recent_genres (avg)          popularity_score
       ↓                            ↓
 Dense layers               Dense layers
(256 → 128 → 64)           (256 → 128 → 64)
       ↓                            ↓
  User Embedding              Item Embedding
       (d=64)                      (d=64)
            ↘                ↙
         dot product / cosine similarity
                    ↓
             Similarity Score
```

Training: **In-batch negative sampling** with batch softmax loss.

#### Key Interview Points for Recommendations

1. **Why two-stage pipeline?** Running a complex ranking model on 10M+ items is infeasible. ANN search on embeddings narrows to 1000 candidates quickly, then rich model ranks precisely. Balances quality and latency.
2. **Explore-exploit trade-off:** Contextual bandits (LinUCB or Thompson Sampling) for exploration. Show new items to some users via epsilon-greedy.
3. **Filter bubbles:** Enforce diversity constraints in re-ranking, monitor catalog coverage as A/B guardrail metric.
4. **Cold start (new user):** Onboarding flow → context-based recommendations → popularity fallback → quick learning from early interactions (contextual bandits)
5. **Cold start (new item):** Content-based embedding from item features → exploration boost (UCB) → similar item embedding bootstrapping
6. **Offline evaluation:** NDCG@K, MAP@K, Recall@K, Hit Rate@K
7. **Online evaluation:** CTR, Watch Rate, Session Length, Return Rate (A/B test)
8. **Scale:** 100M DAU, 50 req/user/day = 60K RPS → 100 API pods, Redis cluster (1TB), 10 FAISS servers, 50 Triton GPU servers

#### Features

- **User features:** account age, preferred genres, watch count 30d, session velocity
- **Item features:** genre, director, avg_rating, release_year, popularity_score
- **Context features:** time of day, device type, location
- **Cross features:** user-genre affinity, user-director affinity, user-item similarity

---

### 4.2 Fraud Detection

**Reference:** Payment processing, banking, e-commerce  
*Source: shafaypro/CrackingMachineLearningInterview:system_design/fraud_detection.md*

#### Requirements

| Requirement | Target |
|-------------|--------|
| Latency (p99) | < 100ms |
| Throughput | 10,000 TPS |
| Availability | 99.99% |
| False Positive Rate | < 0.5% |
| Fraud Detection Rate | > 80% of fraud dollars |
| Scale | 500M transactions/day |

#### Architecture: Hybrid Rules + ML

```
Transaction Event → Kafka → Feature Service → ML Model → Decision
         ↓                        ↓
   < 100ms total         Online Feature Store (Redis)

Decision Logic:
- P > 0.9: Auto-decline
- 0.5 < P < 0.9: Hold for review / 3DS challenge
- P < 0.5: Auto-approve (with monitoring)
```

#### Model: Why Gradient Boosting (LightGBM)

| Factor | Justification |
|--------|---------------|
| Performance | State-of-the-art on tabular data |
| Latency | Inference < 5ms on CPU |
| Interpretability | SHAP values for feature importance |
| Class imbalance | Built-in `scale_pos_weight` |
| Handles missing values | Native handling |

#### Key Features (Velocity-Based)

```python
features = {
    # Velocity features (sliding windows)
    "txn_count_1h": count_transactions(user_id, window="1h"),
    "txn_count_24h": count_transactions(user_id, window="24h"),
    "unique_merchants_24h": count_unique(user_id, "merchant_id", window="24h"),
    "unique_countries_7d": count_unique(user_id, "country", window="7d"),

    # Amount anomalies
    "amount_deviation": (amount - user_avg_amount) / user_std_amount,
    "is_round_number": amount % 10 == 0,

    # Behavioral anomalies
    "unusual_hour": 1 if transaction.hour in [2, 3, 4, 5] else 0,
    "new_country": transaction.country not in user_frequent_countries,
    "new_device": not is_known_device(transaction.device_fingerprint),

    # Network risk
    "vpn_detected": ip_info.vpn_detected,
    "country_mismatch": transaction.ip_country != user.home_country,
    "ip_velocity": count_unique_users_from_ip(transaction.ip, window="1h"),
}
```

#### Key Interview Points for Fraud Detection

1. **Why gradient boosting over deep learning?** Trains faster, lower latency inference, more interpretable (SHAP), better on tabular data.
2. **Label delay problem:** Fraud is only confirmed when chargebacks arrive (30-60 days later). Train on data where labels are confirmed; accept that recent data cannot be used until labels arrive.
3. **Data leakage prevention:** Use temporal splits. Use point-in-time correct feature values. Never include post-transaction information.
4. **Handling class imbalance:** Use `scale_pos_weight` + threshold tuning. Avoid SMOTE — synthetic fraud patterns don't reflect real fraud behavior.
5. **Retraining triggers:** Fraud detection rate drops > 5% (weekly), PSI > 0.25 on top features (monthly), new fraud pattern from analysts, scheduled monthly.
6. **Monitoring:** Alert rate, auto-decline rate, review queue length, model latency (p50/p95/p99), feature compute failures.
7. **Adversarial adaptation:** Fraudsters adapt. Monitor distribution shifts, use anomaly detection layers, behavioral analytics, analyst feedback loops.

#### Evaluation Metrics

| Metric | Formula | Target |
|--------|---------|--------|
| **Recall@FPR0.5%** | Fraud caught at 0.5% false positives | > 80% |
| **Fraud Dollar Recall** | Fraud $ caught / Total fraud $ | > 85% |
| **AUC-ROC** | Area under ROC curve | > 0.97 |
| **AUC-PR** | Area under Precision-Recall | > 0.50 |

*Do NOT use accuracy* — 99.9% accuracy by predicting "no fraud" catches zero fraud.

---

### 4.3 Search Ranking

**Reference:** Google, LinkedIn, Elasticsearch, e-commerce search  
*Source: shafaypro/CrackingMachineLearningInterview:system_design/README.md*

#### Learning to Rank Paradigm

| Approach | Description | Use When |
|----------|-------------|----------|
| **Pointwise** | Predict relevance score per doc independently | Simple baseline |
| **Pairwise** | Predict which of two docs is more relevant | Better than pointwise |
| **Listwise** | Optimize directly on ranking metrics (NDCG, MAP) | Best quality — SOTA |

#### Feature Categories

- **Query features:** query length, query type (informational vs navigational)
- **Document features:** PageRank, freshness, content quality score
- **Query-document features:** BM25, embedding similarity, historical CTR for this query-doc pair
- **User features:** search history, location, device type, query reformulations

#### Architecture

```
Query Input
   ↓
Candidate Retrieval:
├── Dense retrieval (bi-encoder, ANN search) — semantic matching
└── Sparse retrieval (BM25, inverted index) — keyword matching

Merge: top 1000 candidates

Ranking:
└── LambdaMART (gradient boosting for ranking) or LambdaRank
    Input: query features + doc features + query-doc features
    Output: relevance score per document

Re-ranking (top 100):
└── BERT cross-encoder
    Slower but higher quality — uses full query × doc interaction

Final: Top 10-20 results
```

#### Evaluation

- **Offline:** NDCG@10, MAP, MRR (Mean Reciprocal Rank)
- **Online:** CTR, Session success rate (user found what they needed), zero-click rate

---

### 4.4 Ad Click Prediction

**Reference:** Google Ads, Meta Ads, programmatic advertising

#### Problem

Predict probability a user will click on an ad given:
- User profile + context
- Ad content + advertiser
- Placement context (page, position)

#### Model: Wide & Deep (Google) / DLRM (Meta)

```
Input:
├── Sparse features: user_id, ad_id, publisher_id → embeddings
└── Dense features: age, CTR history, bid price, time of day

Wide component: Linear model on cross-product feature interactions
                (memorization of frequent co-occurrences)

Deep component: Feed-forward NN on embeddings
                (generalization to unseen feature combos)

Output: P(click | user, ad, context) → used to rank ads by Expected Value = P(click) × bid
```

#### Key Considerations

- **Latency:** < 50ms for ad serving (hundreds of candidates scored simultaneously)
- **Scale:** Billions of ad impressions/day
- **Business metric:** Revenue = Σ (bid × P(click)) for winning ads
- **Exploration:** Need some exploration for new ads (no click history)
- **Position bias:** Clicks are biased by ad position — use Inverse Propensity Scoring (IPS) to debias

---

### 4.5 Content Moderation at Scale

**Reference:** Facebook, Twitter/X, YouTube, TikTok  
*Source: shafaypro/CrackingMachineLearningInterview:system_design/README.md*

#### Requirements

- Detect harmful content: hate speech, NSFW, misinformation, spam
- Scale: 10M+ posts per day
- Latency: < 200ms for real-time decisions

#### Architecture: Fast Path + Slow Path

```
Post Upload
    ↓
FAST PATH (< 50ms):
├── Rule-based filters (known slurs, banned links)
└── Lightweight ML model (DistilBERT quantized / LightGBM on TF-IDF)

Output: Auto-approve (low risk) / Auto-remove (high risk) / Hold for slow path

SLOW PATH (async, seconds):
├── Heavy model (fine-tuned BERT for text, ResNet for images)
├── Multi-modal model for multimedia (video + audio + text)
└── Human review queue for borderline cases

Appeals process: Users can contest decisions
```

#### Models

- Multi-label classification — a post can be multiple violation types simultaneously
- Binary classifiers per category: hate, NSFW, spam, misinformation (each tuned separately)
- Ensemble of text + image models for multimedia content

#### Key Considerations

1. **Class imbalance:** Most content is legitimate. Use weighted loss, oversampling of positive class.
2. **Language diversity:** Multi-lingual model (mBERT, XLM-R) or per-language models.
3. **Adversarial users:** Bypass attempts (character substitution, l33tspeak, image-based text).
4. **Human-in-the-loop:** Low-confidence predictions → human review queue → human labels → training data for model improvement.
5. **Monitoring:** False positive rate (legitimate content blocked), false negative rate (harmful content missed), human review queue length.

---

### 4.6 Spam Classifier

**Architecture:**
- **Baseline:** Logistic Regression on TF-IDF
- **Production:** Gradient Boosting + text embeddings (sentence-transformers)
- **Premium:** Fine-tuned BERT for semantic spam detection

**Key Features:** TF-IDF of subject/body, sender domain reputation, email header features (SPF, DKIM), user-sender relationship, reply/engagement history

**Latency:** < 100ms for real-time email filtering (pre-compute sender reputation)

---

### 4.7 Anomaly Detection

**Use cases:** Server metrics, financial transactions, IoT sensors, model performance monitoring

#### Approaches by Complexity

| Approach | When to Use | Notes |
|----------|-------------|-------|
| Z-score, IQR | Simple, stationary data | Baseline, easy to explain |
| ARIMA residuals | Time-series with trend/seasonality | Statistical, interpretable |
| Isolation Forest, LOF | Multivariate tabular data | Unsupervised ML |
| LSTM Autoencoder | Complex temporal patterns | Learns "normal", flags high reconstruction error |
| Forecasting + threshold | When you can predict expected value | Most intuitive for ops teams |

**Pipeline:**
1. Rolling window features: mean, std, min, max over last N periods
2. Seasonal decomposition (STL) to separate trend, seasonality, residual
3. Dynamic thresholding on residuals (adapts as system behavior evolves)
4. Alert deduplication (correlate alerts from related metrics)

---

### 4.8 Feature Store Design

**Reference:** Feast, Tecton, Databricks Feature Engineering, Hopsworks  
*Source: kengio/databricks-certification-study-guide:shared/interview-prep/14-ml-system-design.md*

#### Entity-Centric Design (Key Principle)

```
Instead of feature tables per model, create them per business entity:

ml.features.user_demographics        — keyed on user_id, updated daily
ml.features.user_transactions        — keyed on user_id, updated hourly
ml.features.product_interactions     — keyed on user_id + product_id, updated hourly

Each model creates its training set via feature lookups:
training_set = fe.create_training_set(
    df=labels_df,  # has user_id, label, timestamp
    feature_lookups=[
        FeatureLookup(table_name="ml.features.user_demographics", lookup_key="user_id"),
        FeatureLookup(table_name="ml.features.user_transactions", lookup_key="user_id"),
    ],
    label="churned",
)
```

#### Online vs Offline Store

| Dimension | Offline Store | Online Store |
|-----------|---------------|--------------|
| **Storage** | Data lake (S3, HDFS), Delta Lake | Redis, DynamoDB, Cassandra |
| **Latency** | Minutes to hours | Sub-millisecond |
| **Use** | Training, batch inference | Real-time serving |
| **Update freq** | Daily/hourly batch | Real-time streaming or point-in-time writes |
| **Scale** | Petabytes | Gigabytes (hot features only) |

#### Point-in-Time Correctness

For time-sensitive features, use **point-in-time lookups** with a timestamp column to avoid data leakage — the feature store only joins features available *before* the label timestamp.

```python
training_set = fe.create_training_set(
    df=labels_df,                    # includes event_timestamp column
    feature_lookups=[FeatureLookup(...)],
    label="churned",
    exclude_columns=["user_id"]
    # Automatically handles point-in-time joins!
)
```

#### Key Interview Points for Feature Stores

1. **Wide vs narrow tables:** Wide tables (one table, many features) vs many narrow entity tables — prefer narrow for reusability, wide for query efficiency.
2. **Feature versioning:** When feature definition changes, version it to avoid training-serving skew.
3. **Feature computation:** Decouple from model training — if `user_transactions` needs hourly refresh, that's one shared job, not one per model.
4. **Online publishing:** Only publish features needed for real-time serving (not all features) to keep online store lean.

---

### 4.9 ML Pipeline Architecture

*Source: shafaypro/CrackingMachineLearningInterview:system_design/README.md*

#### End-to-End Architecture

```
RAW DATA SOURCES
├── Transactional DB (PostgreSQL, MySQL)
├── Event streams (Kafka)
├── Data warehouse (Snowflake, BigQuery)
└── Third-party APIs
        ↓
ETL PIPELINE
├── Ingestion (Spark, Airflow)
├── Transformation (dbt, Spark)
├── Validation (Great Expectations)
└── Storage (S3/GCS parquet, Delta Lake)
        ↓
FEATURE PIPELINE
├── Batch features → Offline Feature Store
├── Streaming features → Online Feature Store (Redis)
└── Feature Registry (Feast/Tecton)
        ↓
MODEL TRAINING
├── Experiment Tracking (MLflow / W&B)
├── Distributed training (PyTorch DDP, DeepSpeed)
├── Cloud (SageMaker, Vertex AI, Azure ML)
└── Model Registry (staging → production lifecycle)
        ↓
MODEL SERVING
├── REST API (FastAPI + Docker + Kubernetes)
├── gRPC (lower latency, binary protocol)
├── Batch prediction jobs (Spark, Airflow)
└── Serving Frameworks (Triton, TorchServe, BentoML)
        ↓
MONITORING
├── Infrastructure: Latency (p50/p95/p99), Throughput, Error rate
├── Data/Model: Feature drift (PSI, KS test), Prediction distribution shift
├── Business KPIs
└── Tools: Grafana + Prometheus, Evidently AI, Arize AI

```

---

### 4.10 A/B Testing Infrastructure

*Source: shafaypro/CrackingMachineLearningInterview:system_design/README.md*

#### Setup

1. Define the **primary metric** (CTR, conversion, session time)
2. Define **guardrail metrics** (must not regress — catalog coverage, diversity)
3. Compute **MDE** (Minimum Detectable Effect) to size the experiment
4. Randomly assign users to **control** (old) and **treatment** (new) — consistent assignment
5. Run for minimum **1-2 full business cycles** (capture weekly seasonality)
6. Use t-test / z-test for continuous metrics; Fisher's exact / Chi-squared for binary metrics
7. Set **α = 0.05**, power **1-β = 0.80**
8. Correct for multiple comparisons if testing multiple metrics

#### Common Pitfalls

| Pitfall | Description | Solution |
|---------|-------------|----------|
| **Novelty effect** | Users engage more with new things initially | Run for ≥ 2 weeks |
| **Peeking problem** | Stopping early because p < 0.05 at one point | Sequential testing or fixed horizon |
| **Network effects** | Social systems — control users influenced by treatment | Cluster-based randomization |
| **Sample ratio mismatch** | Actual split ≠ intended split | Verify before analyzing |
| **Long-term harm** | Short-term CTR boost hides long-term satisfaction drop | Add 30/90-day cohort metrics |

#### Champion/Challenger Pattern (for Model Retraining)

```python
import mlflow

champion_model = mlflow.pyfunc.load_model("models:/ml.prod.recommender@champion")
candidate_model = mlflow.pyfunc.load_model(f"models:/ml.prod.recommender/{new_version}")

champion_auc = evaluate(champion_model, holdout_df)
candidate_auc = evaluate(candidate_model, holdout_df)

if candidate_auc > champion_auc + 0.005:  # require meaningful improvement
    client.set_registered_model_alias("ml.prod.recommender", "champion", new_version)
```

---

## 5. GenAI/LLM System Design Topics (2026-Specific)

### 5.1 RAG System Design

**Reference:** Perplexity AI (200M daily queries), Slack AI, DoorDash chatbot  
*Source: shafaypro/CrackingMachineLearningInterview:system_design/ml_system_design_patterns.md; huyenchip.com/2024/07/25/genai-platform.html*

#### When to Use RAG

- Q&A over private/proprietary knowledge bases
- Document search and synthesis
- Customer support automation
- Code assistance with private codebase context
- Any system where the LLM needs current/domain-specific information

#### Full RAG Architecture

```
INDEXING PIPELINE (Offline)
Documents → Chunker → Embedder → VectorDB + Metadata Store

SERVING PIPELINE (Online)
User Query
    ↓
[Query Rewriter]          ← optional (expand, clarify, decompose)
    ↓
[Hybrid Search]           ← BM25 (keyword) + Dense (semantic) — always use hybrid in production
    ↓
[Reranker]                ← cross-encoder reranking (Cohere Rerank v3, BGE-Reranker)
    ↓
[Context Assembly]        ← select top K chunks, order for least-in-middle effect
    ↓
[LLM Generation]          ← "Answer ONLY from the provided context"
    ↓
[Answer + Citations]      ← source attribution, verifiability
```

#### Key Design Decisions

| Decision | Options | Production Recommendation |
|----------|---------|--------------------------|
| **Chunking** | Fixed, recursive, semantic | Recursive for general; semantic for technical docs |
| **Embedding model** | OpenAI, Voyage, Cohere | Voyage-3 for highest quality; OpenAI text-embedding-3 for cost |
| **Vector DB** | Pinecone, Weaviate, pgvector, Qdrant | Weaviate for hybrid; pgvector if already on Postgres |
| **Search** | Dense only, hybrid | **Always hybrid in production** |
| **Reranking** | None, cross-encoder, Cohere | Cohere Rerank v3 for best quality |
| **Generation model** | Any LLM | Claude Sonnet for quality/cost balance |

#### Perplexity AI's RAG Pipeline (200M Queries/Day)

5-stage pipeline verified by ByteByteGo analysis:
1. **Query Intent Parsing:** LLM parses semantic intent beyond keyword level
2. **Live Web Retrieval:** Real-time search index — information always fresh
3. **Snippet Extraction:** Extract most relevant chunks (not full pages)
4. **Synthesized Answer with Citations:** "Answer ONLY from what you retrieved" — all answers cite sources
5. **Conversational Memory:** Multi-turn context management

**Infrastructure:** Vespa.ai for the vector store (handles hybrid search at scale), fine-tuned Sonar models for generation.

#### RAG Scaling Bottlenecks

```
Bottleneck 1: Embedding at indexing time
→ Batch API calls (1000 texts/request)
→ Cache embeddings for unchanged documents
→ Cheaper model for initial indexing

Bottleneck 2: Vector search latency
→ HNSW index (lower latency than IVF)
→ Pre-filter by metadata before vector search (reduces search space 10-100x)
→ Cache frequent queries (Redis with TTL)

Bottleneck 3: LLM generation latency
→ Streaming responses (user sees text appear)
→ Route simple queries to Haiku; complex to Sonnet/Opus
→ Reduce context size (fewer, better chunks)
```

#### Diagnosing RAG Failures

```
Low faithfulness (hallucinations):
→ Check: Does context actually contain the answer?
  → If NO: retrieval problem
    - Wrong chunks retrieved → better embedding, reranking
    - Relevant chunks missed → increase top-k, hybrid search
    - Chunk boundary cuts key info → better chunking
  → If YES: generation problem
    - Strengthen prompt: "Answer ONLY from the provided context"
    - Context too long (lost-in-middle) → reduce to top-3 chunks
    - Try larger model

Low relevance (off-topic answers):
→ Better query rewriting
→ Improve chunking strategy
→ Add metadata filters
```

#### Interview Question: Design a Document Q&A Assistant

```
Requirements: 300,000 legal contracts, enterprise users, PII concerns

Architecture:
1. Chunking: Recursive splitting with 512-token chunks, 50-token overlap
2. Embedding: Voyage Law-2 (domain-specific legal embeddings)
3. Storage: Weaviate (hybrid search) + S3 (raw documents)
4. Retrieval: Hybrid BM25 + dense, top-20, then Cohere rerank to top-5
5. Privacy: LLM processing in VPC (no data leaves), PII masking before embedding
6. Generation: Claude Sonnet (balanced quality/cost) with strict citation requirement
7. Evaluation: Human annotators rate faithfulness (1-3), relevance (1-3) on 500 golden Q&A pairs

Cost estimate: ~300K contracts × avg 10 pages × $0.02/1K tokens embedding = ~$6,000 one-time indexing
Per query: ~$0.005-0.02 depending on context length and model tier
```

---

### 5.2 ChatGPT-Like System Design

**Reference:** OpenAI ChatGPT, Anthropic Claude, Google Gemini  
*Source: alexeygrigorev/ai-engineering-field-guide:interview/questions/04-ai-system-design.md; huyenchip.com genai-platform*

#### Core Platform Components

```
USER REQUEST
    ↓
[API Gateway] — authentication, rate limiting, routing
    ↓
[Context Manager]
├── System prompt (instructions, persona)
├── Conversation history (truncated to fit context window)
├── RAG-retrieved context (if knowledge base)
└── User message
    ↓
[Model Router / Gateway]
├── Simple queries → fast/cheap model (Haiku)
├── Complex reasoning → large model (Sonnet/Opus)
├── PII redactor (Uber GenAI Gateway pattern)
└── Content safety filter
    ↓
[LLM API]
├── Third-party: OpenAI, Anthropic, Google
└── Self-hosted: vLLM, TGI on K8s (for privacy/cost at scale)
    ↓
[Post-processing]
├── Output guardrails (safety filter, PII removal)
├── Citation attachment (if RAG)
└── Streaming response to user
    ↓
[Feedback & Memory]
├── Implicit signals: thumbs up/down, edit behavior, session length
├── Cross-conversation memory: user preferences, facts about user
└── Evaluation flywheel: feedback → eval dataset → model improvement
```

#### Key Design Decisions for ChatGPT-Like Systems

1. **Context window management:** How to handle conversations that exceed the context limit?
   - Options: truncation (lose history), summarization, sliding window, retrieval-based memory
   
2. **Memory architecture:** Session vs persistent vs associative
   ```
   Short-term: in-context (last N turns)
   Long-term: retrieved memory (user facts, past conversation summaries)
   Associative: knowledge graph (for complex multi-hop queries)
   ```

3. **Model serving at scale:**
   - vLLM for high-throughput inference (continuous batching, PagedAttention for KV cache)
   - KV cache sharing for common system prompt prefixes (prompt prefix caching)
   - Speculative decoding (draft model generates tokens, large model verifies)

4. **Cross-conversation memory (actual interview Q from real company):**
   ```
   Design:
   1. After each conversation: LLM extracts key user facts (preferences, corrections made)
   2. Store in user memory store (vector DB keyed by user_id)
   3. At start of new conversation: retrieve top-K relevant memories
   4. Inject into system prompt: "What you know about this user: ..."
   Privacy: user opt-out, memory deletion endpoint, data retention policy
   ```

---

### 5.3 Agent System Design

**Reference:** Doctolib healthcare agents (~17K daily messages), Anthropic multi-agent research  
*Source: alexeygrigorev/ai-engineering-field-guide:interview/questions/01-theory.md and 04-ai-system-design.md; shafaypro/ml_system_design_patterns.md*

#### What Makes a System "Agentic"

An agent is a system that:
- **Observes** context
- **Decides** what to do (planning/reasoning)
- **Acts** toward a goal using tools, memory, and feedback loops
- **Iterates** — each action informs the next step

#### Core Agent Architecture

```
User Input
    ↓
[Task Planner / Router]
    ↓
┌────────────────────────┐
│      Agent Loop         │
│                         │
│  [LLM] → think → [Tool]│
│    ↑                ↓  │
│    └──── observe ───┘  │
│                         │
│  Until: task complete   │
│     or: max iterations  │
└────────────────────────┘
    ↓
[Output Validator]
    ↓
[Human Review?] (optional HITL)
    ↓
Final Response / Action
```

#### Tool Design Principles

```python
@tool
def search_knowledge_base(query: str, max_results: int = 5) -> list[dict]:
    """
    Search the internal knowledge base for relevant information.
    
    Args:
        query: Natural language search query
        max_results: Number of results to return (1-10)
    
    Returns:
        List of {title, content, relevance_score} dicts
    """
    # Clear description → LLM knows WHEN to use this tool
    # Typed inputs → LLM knows HOW to call this tool
    # Return type → LLM can interpret results correctly
```

**Bad tool design:** Vague names, no descriptions, untyped inputs.

#### Multi-Agent vs Single-Agent Decision Tree

```
Is the task parallelizable?
├── Yes → Multi-agent (fan-out pattern)
└── No → Single agent is usually better

Does it require multiple specialized domains?
├── Yes (legal + technical + financial) → Multi-agent  
└── No → Single agent

Is total token count within 200K?
├── Yes → Single agent (simpler, cheaper)
└── No → Multi-agent (divide and conquer)

Is the team willing to debug multi-agent coordination?
├── Yes → Multi-agent if needed
└── No → Single agent with good tools
```

#### Key Interview Questions for Agents

| Question | What Interviewers Want to Hear |
|----------|-------------------------------|
| What makes a system agentic? | LLM + tools + memory + multi-step planning loop |
| When is an agent the *wrong* solution? | Task is deterministic, doesn't require planning, scripted workflow would work |
| How do you detect/stop infinite loops? | Max iterations, termination conditions, circuit breakers |
| How do you sandbox tool execution? | Container isolation, permission scoping, output validation before execution |
| Biggest security risks with agents? | Prompt injection (malicious content in tool output hijacks agent), data leakage, unsafe code execution |
| How do you monitor agent behavior in production? | Trace all LLM calls and tool invocations, LLM-as-judge for output quality, alert on unusual tool call patterns |

#### Production Agent Examples

| Company | System | Scale | Architecture |
|---------|--------|-------|--------------|
| **Doctolib** | Healthcare customer support | ~17K daily messages | Specialized agents in directed graph |
| **Anthropic** | Multi-agent research | ~15x more tokens than chat | Opus orchestrator + Sonnet subagents |
| **LinkedIn** | AI agent platform | Production scale | Strict data layer siloing |
| **DoorDash** | LLM chatbot evaluation flywheel | Production scale | Hierarchical RAG + simulation eval |
| **Airbnb** | Conversational AI | Production scale | Chain of Thought reasoning + guardrails |

#### Design Question: Multi-step Agentic Workflow (Meeting Scheduling)

```
Requirements: Agent schedules meetings by checking calendars, sending invites, handling conflicts

Architecture:
1. Intent classification: Meeting scheduling? Route to scheduler agent
2. Tools:
   - read_calendar(person_id, date_range) → available slots
   - check_preferences(person_id) → preferences, timezone, constraints
   - create_calendar_event(participants, time, details) → confirmation
   - send_message(to, content) → email/Slack notification
3. Loop: Parse attendees → check calendars → find overlap → propose slots → confirm → create event
4. HITL: Ambiguous time preferences → ask user to confirm
5. Termination: Event created (success) or max 5 attempts (failure, escalate to human)
6. Monitoring: Trace all LLM calls with timestamps, tool success/failure rates, user satisfaction rating
```

---

### 5.4 Real-Time ML Serving at Scale

*Source: shafaypro/CrackingMachineLearningInterview:system_design/ml_system_design_patterns.md*

#### Architecture Pattern

```
Request → [API Gateway]
               ↓
         [Feature Store (Redis)] ← [Real-time events (Kafka)]
               ↓
         [Model Server]
         (TorchServe / vLLM / Triton Inference Server)
               ↓
         [Post-processing] (threshold, ranking, safety filter)
               ↓
         Response (target: < 100ms total)
```

#### Latency Budget (Fraud Detection example — 100ms SLA)

```
Network in:         5ms
Feature fetch:     10ms  (Redis online store)
Model inference:   20ms  (GPU-accelerated LightGBM or LLM)
Post-processing:    5ms
Network out:        5ms
─────────────────────
Total:             45ms  (55ms safety margin)
```

#### Serving Framework Comparison

| Framework | Best For | Notes |
|-----------|----------|-------|
| **Triton Inference Server** | Multi-framework (TF, PyTorch, ONNX) | NVIDIA, production standard for GPU serving |
| **vLLM** | LLM inference | PagedAttention, continuous batching, highest throughput |
| **TorchServe** | PyTorch models | Native PyTorch, good for custom handlers |
| **TF Serving** | TensorFlow models | Google's production standard |
| **BentoML / Seldon** | Framework-agnostic | Good for mixed model types |
| **KServe** | Kubernetes-native | Best for K8s deployments |

#### Optimization Techniques

```
Model-level:
├── Quantization: FP32 → INT8 (2-4x speed, ~1% accuracy drop)
├── ONNX export: cross-platform inference
├── TensorRT: NVIDIA GPU optimization (2-5x speedup)
└── Model distillation: large model → small fast model (10x smaller)

System-level:
├── Request batching: batch multiple requests per inference (GPU efficiency)
├── Caching: semantic cache (Redis) for repeated queries
├── Model warm-up: avoid cold starts with persistent connections
├── Horizontal scaling: multiple model replicas behind load balancer
└── Precomputed predictions: batch inference for all users daily
```

---

### 5.5 Fine-Tuning Infrastructure Design

*Source: alexeygrigorev/ai-engineering-field-guide:interview/questions/01-theory.md*

#### When to Fine-Tune vs RAG vs Prompt Engineering

| Approach | Use When | Cost | Iteration Speed |
|----------|----------|------|-----------------|
| **Prompt Engineering** | Model needs behavioral guidance | Very low | Very fast |
| **RAG** | Model needs external/current facts | Low-medium | Fast |
| **Fine-tuning (PEFT/LoRA)** | Domain adaptation, consistent style, private data can't leave | Medium | Slow |
| **Full Fine-tuning** | Complete behavior change, research | High | Slow |
| **Pretraining from scratch** | Novel domains with massive data | Very high | Very slow |

#### PEFT/LoRA Infrastructure

```
Training Infrastructure:
├── LoRA: Add low-rank adapter matrices to transformer layers
│   - Only train 0.1-1% of parameters
│   - Merge adapters into base model for inference
│   - Multiple LoRA adapters for different tasks (hot-swap at serving time)
│
├── Quantization (QLoRA): 4-bit base model + LoRA fine-tuning
│   - 10x memory reduction vs full fine-tuning
│   - Enables fine-tuning 70B models on single GPU
│
└── Distributed training: FSDP, DeepSpeed ZeRO-3 for large models

Evaluation:
├── Benchmark on holdout set with task-specific metrics
├── LLM-as-judge for open-ended quality
├── A/B test fine-tuned vs base model in production
└── RLHF: human preferences as training signal
```

#### Converting User Behavior to Training Signals

*Source: alexeygrigorev/ai-engineering-field-guide:interview/questions/01-theory.md*

```
Implicit signals (automatic):
├── User edits AI output → "the original was wrong/suboptimal"
├── User accepts suggestion → positive signal
├── User rejects suggestion → negative signal
├── Session length after response → engagement signal
└── User copy-pastes response → high quality signal

Explicit signals (user-initiated):
├── Thumbs up/down
├── Star ratings
└── Detailed feedback forms

Signal collection:
→ Log all signals with (query, response, signal, user_id, timestamp)
→ Use as DPO (Direct Preference Optimization) training data
→ Annotate borderline cases with human labelers
```

---

### 5.6 Vector Search Systems at Scale

*Source: shafaypro/CrackingMachineLearningInterview:system_design/ml_system_design_patterns.md; huyenchip.com genai-platform*

#### ANN Algorithm Comparison

| Algorithm | Recall | Build Time | Index Size | Best For |
|-----------|--------|------------|------------|----------|
| **HNSW** | High | Slow | Large | Production (best recall/latency tradeoff) |
| **IVF** | Medium-High | Fast | Medium | When build speed matters |
| **ScaNN** | High | Medium | Medium | Google's production system |
| **FAISS** | High | Fast | Medium | Meta's production system, on-premise |
| **Annoy** | Medium | Fast | Small | Spotify, memory-constrained environments |

#### Scaling to 10M+ Documents

```
Indexing:
├── Parallel embedding generation (batch API + multiple workers)
├── Pinecone serverless or Weaviate cluster (horizontal scaling)
└── Incremental updates vs full re-index strategy

Query time:
├── Pre-filter by metadata (reduces search space 10-100x)
├── Caching: Redis cache for frequent queries (TTL=1h)
├── HNSW index (sub-linear query time)
└── Namespace isolation for multi-tenancy

Multi-tenancy:
├── Separate vector spaces per tenant (isolation + security)
├── Shared index with metadata filters (cost-efficient but shared infra)
└── Hybrid: large tenants get dedicated, small tenants share

Cost tiering:
├── Hot docs: main HNSW index (fast, expensive)
├── Cold docs: compressed index or re-embed on demand (slow, cheap)
└── Archive: not indexed, only full-text search fallback
```

#### Hybrid Search Implementation

```
Query: "machine learning system design interview 2026"
    ↓
BM25 (sparse): keyword matching on "machine learning", "system design", "interview"
    → Returns: 100 candidates with exact keyword matches
    ↓
Dense (semantic): embedding similarity
    → Returns: 100 candidates semantically related
    ↓
Reciprocal Rank Fusion (RRF):
    Final score = Σ 1/(k + rank_i) where k=60 (smoothing constant)
    → Merges both lists, boosts docs appearing in both
    ↓
Cross-encoder reranker:
    → Full interaction between query and top-20 candidates
    → Most accurate but slow (only use on top-K)
```

---

### 5.7 LLM Cost & Latency Optimization

*Source: alexeygrigorev/ai-engineering-field-guide:interview/questions/01-theory.md; shafaypro/ml_system_design_patterns.md*

#### Cost Reduction Strategies (10x Reduction)

```
1. Model selection (biggest lever):
   Claude Haiku vs Opus = 30-60x cost difference
   → Route simple tasks to Haiku, complex to Opus
   
   Approximate 2026 pricing:
   - Haiku:  $0.00025/1K input,  $0.00125/1K output
   - Sonnet: $0.003/1K input,   $0.015/1K output
   - Opus:   $0.015/1K input,   $0.075/1K output

2. Prompt optimization:
   → Remove redundant instructions
   → Efficient system prompts (no verbose repetition)
   → Structured output (JSON) is often shorter than prose

3. Caching:
   → Exact cache: Redis for identical queries
   → Semantic cache: cache by embedding similarity (TTL=1h)
   → Prefix caching: Anthropic/OpenAI support shared prompt caching

4. Output length control:
   → Set max_tokens precisely for each task type
   → Streaming enables early termination
   → Structured output limits generation length

5. Batching:
   → Batch multiple requests into one API call
   → Asynchronous processing for non-real-time tasks

6. Smart routing classifier:
   complexity_score = small_classifier.predict(prompt)
   if score < 0.3: use Haiku   # 70% of queries
   elif score < 0.7: use Sonnet # 20% of queries
   else: use Opus              # 10% of queries — 10x overall cost reduction
```

#### Cost Estimation (Interview Example)

```
AI Code Assistant: 100K daily developers × 10 interactions/day
= 1M interactions/day
Each: ~1K input tokens + ~1K output tokens
= 1B input + 1B output tokens/day

Using GPT-4 Turbo ($0.003/1K input, $0.01/1K output):
Input: 1B × $0.003 = $3,000/day
Output: 1B × $0.01 = $10,000/day
Total: $13,000/day = $390K/month

Cost reduction strategies:
→ Route 70% to Haiku → $390K × 0.3 = $117K/month (70% reduction)
→ Semantic caching for FAQ (20% cache hit rate) → additional 20% reduction
→ Prompt compression (remove boilerplate) → 20% token reduction
→ Final: ~$70K/month (82% reduction)
```

---

### 5.8 LLM Safety & Guardrails

*Source: alexeygrigorev/ai-engineering-field-guide:interview/questions/01-theory.md; systemdesignhandbook.com*

#### Threat Model

| Threat | Description | Mitigation |
|--------|-------------|-----------|
| **Prompt injection** | Malicious content in retrieved docs hijacks agent behavior | Separate system prompt from user content, output validation, sandboxing |
| **Jailbreaking** | Crafted prompts bypass safety instructions | Constitutional AI, RLHF, output classifier |
| **Hallucination** | Model generates confident false information | RAG + citation requirement, confidence estimation |
| **Data leakage** | PII in prompts/logs or model memorization | PII redactor before LLM, log sanitization, VPC deployment |
| **Unsafe code execution** | Agent generates and runs malicious code | Sandboxed execution environment (Docker), code review before execution |

#### Guardrail Architecture

```
Input side (before LLM):
├── Content classifier (hate, NSFW, PII, prompt injection detection)
├── Rate limiting (prevent abuse)
└── Input sanitization (remove/mask PII)

Prompt engineering:
├── Clear system boundaries ("You are a customer support agent for X. Do not...")
├── Constitutional AI constraints in system prompt
└── Few-shot examples of correct refusals

Output side (after LLM):
├── Content classifier (same as input — different attack surface)
├── PII detection and removal
├── Factual consistency checker (for high-stakes domains)
└── Sensitive topic detector (escalate to human)

Monitoring:
├── Log all LLM inputs/outputs (redacted)
├── Monitor refusal rate, jailbreak attempts
└── Human review of flagged conversations
```

---

## 6. Key Concepts Deep Dives

### 6.1 Online vs Offline Features

| Dimension | Offline Features | Online Features |
|-----------|-----------------|-----------------|
| **Computation timing** | Pre-computed in batch | Computed at request time |
| **Storage** | Data warehouse, S3 | Redis, DynamoDB, Feature Store online store |
| **Latency** | Pre-computed: <1ms lookup | Real-time compute: 10-100ms |
| **Freshness** | Hours-days stale | Milliseconds to seconds |
| **Examples** | User demographics, 30-day purchase history, item genre | Current session activity, real-time velocity, live pricing |
| **Cost** | Low (compute once) | High (compute per request) |

**Rule of thumb:** Use offline features for stable characteristics (user profile, item metadata), online features for recency-sensitive signals (current session, real-time velocity).

---

### 6.2 Model Serving Patterns

#### Pattern 1: Synchronous REST API
```
Client → POST /predict → Feature fetch → Model inference → Response
Use when: Latency requirement < 1s, interactive use cases
```

#### Pattern 2: Batch Inference
```
Cron/event trigger → Feature fetch for all users → Batch model scoring → Store scores in DB
Client → GET /recommendations → Read pre-computed scores from DB
Use when: Pre-computing predictions for all users is acceptable (recommendations, risk scores)
```

#### Pattern 3: Streaming Inference
```
Events → Kafka → Consumer → Feature computation → Model inference → Output event → Kafka
Use when: Real-time feature pipeline is needed (fraud detection, anomaly detection)
```

#### Pattern 4: Cascading Models
```
Request → Fast/cheap model (< 5ms)
         ↓ if uncertain
         Slow/expensive model (< 100ms)
         ↓ if still uncertain
         Human review queue
Use when: Most requests are easy; only a minority need expensive inference
```

---

### 6.3 Data Pipeline Architecture

#### Lambda Architecture for ML

```
Batch Layer: Historical data → Full model retrain → Precomputed scores for all users
Speed Layer: Real-time events → Lightweight model → Fresh adjustments (recency signals)
Serving Layer: Batch score + Speed adjustment = Final score shown to user
```

**When to use:** Recommendation systems where both historical patterns (batch) and recent behavior (streaming) matter.

#### Kappa Architecture (Streaming-Only)

```
All events → Kafka → Stream processor → Real-time features → Model → Scores
Use when: Simplicity is valued over historical reprocessing; latency < 1 minute
```

---

### 6.4 Monitoring & Alerting

#### Three Levels of Model Monitoring

```
Level 1: Infrastructure Monitoring
├── Latency: p50, p95, p99 (alert if p99 > SLA)
├── Throughput: requests/second
├── Error rate: 4xx, 5xx responses
└── GPU/CPU utilization

Level 2: Data Drift Monitoring
├── Feature drift: PSI (Population Stability Index) > 0.2 = significant
├── Statistical tests: KS test (continuous), Chi-squared (categorical)
├── Input distribution shift (even before labels arrive)
└── Prediction distribution shift

Level 3: Model Performance Monitoring
├── Label-dependent: accuracy, precision, recall (requires ground truth)
├── Proxy metrics: click-through rate, user engagement
├── Business KPIs: revenue impact, customer satisfaction
└── Alert thresholds: > 5% regression from baseline
```

#### Key Monitoring Tools

| Tool | Use Case |
|------|----------|
| **Grafana + Prometheus** | Infrastructure metrics (latency, throughput, errors) |
| **Evidently AI** | Drift detection (open source, production-ready) |
| **Arize AI / WhyLogs** | ML observability, feature monitoring |
| **MLflow** | Experiment tracking, model versioning |
| **Lakehouse Monitoring** (Databricks) | Integrated drift + performance monitoring |
| **PagerDuty / OpsGenie** | Alerting and escalation |

---

### 6.5 Model Retraining Strategies

#### Trigger Types

| Trigger | Description | Use When |
|---------|-------------|----------|
| **Scheduled** | Retrain on fixed cadence (daily, weekly) | Stable domains, safety net |
| **Drift-triggered** | PSI > threshold, score distribution shift | Dynamic environments |
| **Performance-triggered** | Accuracy drops below threshold | When labels arrive promptly |
| **Event-triggered** | Major real-world event (pandemic, policy change) | High-impact domain shifts |
| **Data-triggered** | X new training examples accumulated | Online learning systems |

#### Deployment Strategies

```
1. Shadow mode:     Run new model alongside production, compare outputs (no user impact)
2. Canary:          5% traffic to new model, 48-hour observation window
3. Blue/Green:      100% traffic swap with instant rollback via feature flag
4. A/B test:        50/50 split with statistical significance testing
5. Progressive:     5% → 20% → 50% → 100% based on metric stability
```

---

### 6.6 Scalability Considerations

#### Read-Heavy Systems (Recommendations, Feature Lookup)

- Cache feature lookups (Redis, pre-warm on startup)
- Replicate model serving behind load balancer
- Pre-compute predictions for common inputs

#### Write-Heavy Systems (Training Data, Feedback Collection)

- Async logging and feedback collection (don't block serving)
- Batch writes to storage
- Event-driven architecture (Kafka → batch processing)

#### High-Throughput Inference (Fraud, Ads)

- Horizontal scaling: multiple model replicas
- Request batching: batch multiple requests per GPU inference
- Model distillation: 10x smaller, faster model
- GPU inference servers (vLLM for LLMs, Triton for classical ML)

#### Rough Capacity Planning

```
100M DAU, 50 req/user/day = 5B requests/day = ~60K RPS

For recommendation serving:
└── 100 API pods × 1000 RPS each = 100K RPS capacity (2x safety)
└── Redis cluster: 20 nodes × 50GB = 1TB for user/item features

For fraud detection (10K TPS):
└── 20 ML service pods (4 CPU, 8GB each), auto-scale to 50
└── Redis cluster: 6 nodes, 50GB total (velocity features for 10M users)
```

---

### 6.7 Cost Optimization

#### Compute Optimization

| Technique | Savings | Trade-off |
|-----------|---------|-----------|
| Model quantization (FP32 → INT8) | 2-4x speed, 4x memory | ~1% accuracy drop |
| Model distillation | 10x smaller/faster model | Quality reduction |
| Batch inference | 5-10x GPU efficiency | Latency (not real-time) |
| Model tiering / routing | 3-10x cost reduction | Routing overhead |
| Spot instances | 60-80% cost reduction | Interruption risk |

#### LLM-Specific Optimization

```
1. Caching:
   - Exact cache (Redis): ~20% hit rate for FAQ use cases
   - Semantic cache: ~30-40% hit rate for similar queries
   - Prompt prefix caching (Anthropic): save cost on long system prompts

2. Model tiering:
   - Simple queries → Haiku (~$0.001/query)
   - Standard queries → Sonnet (~$0.01/query)
   - Complex queries → Opus (~$0.05/query)
   - Route 70% to Haiku → 5-10x overall cost reduction

3. Prompt compression:
   - Remove verbose instructions (use clear, concise prompts)
   - Avoid unnecessary examples in few-shot
   - Use structured prompts (less tokens than prose)
```

---

### 6.8 Latency Requirements

#### Industry Standards by Use Case

| System | P99 Latency Target | Notes |
|--------|--------------------|-------|
| Fraud detection | < 100ms | Before transaction authorization |
| Ad click prediction | < 50ms | Before ad impression |
| Search ranking | < 200ms | User expects instant |
| Recommendation API | < 200ms | Homepage load |
| Content moderation fast path | < 50ms | Before post goes live |
| RAG chatbot | < 3s (first token < 1s) | Streaming acceptable |
| Agent workflows | 5-60s | Background task acceptable |
| Batch predictions | Hours | Pre-computed |

---

## 7. Common Interview Questions (2026)

### Classic ML System Design

| Question | Companies Known to Ask |
|----------|------------------------|
| Design a movie/YouTube recommendation system | Netflix, YouTube, Meta, Snap, Pinterest |
| Design a fraud detection system | Stripe, PayPal, Visa, banks |
| Design a search ranking system | Google, LinkedIn, Amazon, Yelp |
| Design an ad click prediction system | Meta, Google, TikTok |
| Design a spam email classifier | Google, Yahoo, Microsoft |
| Design a real-time content moderation system | Meta, Twitter, TikTok |
| Design a feature store | Most ML-mature companies |
| Design an A/B testing platform | Every tech company |
| Design a churn prediction system (end-to-end pipeline) | Databricks, Salesforce, SaaS companies |
| Design an anomaly detection system | Uber, Datadog, observability companies |

### GenAI System Design (2026-New)

| Question | Companies Known to Ask |
|----------|------------------------|
| Design an AI chatbot (ChatGPT-like) | OpenAI, Anthropic, Cohere, startups |
| Design a Document Q&A Assistant / RAG system | Doctolib, Sprinter Health, enterprise AI |
| Design an AI co-pilot like GitHub Copilot | GitHub, JetBrains, Cursor |
| Design a fraud detection system (LLM-augmented) | AI-first fintech |
| Design ChatGPT's cross-conversation memory | OpenAI, AI assistant companies |
| Design a multi-step agentic workflow | Eightfold.ai, enterprise AI |
| Design a content/policy violation detection system | Meta, TikTok, AI safety companies |
| Design a Perplexity.ai / real-time LLM search engine | AI-first search startups |
| Design a large-scale image generation pipeline for millions of users | Midjourney, Stability AI, Adobe |
| Design a distributed GPU job queue for 100K+ training jobs | xAI, OpenAI infra, Lambda Labs |
| Scale an AI chat feature to 1M daily users | Any AI product company |
| Your app gets 1M queries/day — how do you optimize cost? | Any AI product company |

---

## 8. Common Mistakes to Avoid

*Source: alexeygrigorev/ai-engineering-field-guide:interview/questions/04-ai-system-design.md*

### In the Interview

1. **Jumping to a solution** without clarifying requirements, constraints, and success criteria
2. **Treating the LLM as a source of truth** instead of grounding with retrieval, tools, or citations
3. **Designing only the happy path** without failure modes, monitoring, or evaluation
4. **Over-indexing on tool names** — "I'd use LangChain" instead of explaining WHY you'd chain retrieval and generation
5. **Ignoring cost and latency** — token budgets, model tiering, caching strategies
6. **Ignoring safety** — prompt injection, data leakage, unsafe tool execution
7. **Putting control flow in prompts** instead of the orchestrator (agentic designs)
8. **Choosing agents because they're exciting**, not because the problem requires autonomy
9. **Insisting LLMs can help with problems they can't** — knowing WHEN NOT to use LLMs signals maturity

### In Design

1. **Training-serving skew** — features computed differently at training vs serving time
2. **Data leakage** — using future information in training features (not using temporal splits)
3. **Not handling cold start** — no strategy for new users/items
4. **Ignoring the label delay problem** — fraud labels arrive 30-60 days late
5. **Over-optimizing for offline metrics** while ignoring online business metrics
6. **No rollback strategy** — no champion/challenger pattern, no canary deployment
7. **Static thresholds** — not adapting alert thresholds as system behavior evolves

---

## 9. Books, Courses & Resources

*Source: alexeygrigorev/ai-engineering-field-guide:awesome.md (curated from 2,445 job descriptions + practitioner interviews)*

### Essential Books (2024-2026)

| Book | Author | Focus |
|------|--------|-------|
| **Designing Machine Learning Systems** (O'Reilly, 2022) | Chip Huyen | Classic ML systems, full lifecycle |
| **AI Engineering** (O'Reilly, 2025) | Chip Huyen | Foundation models, RAG, agents, evals |
| **Machine Learning System Design Interview** | Alex Xu, Sahn Lam | Interview-focused, practical examples |
| **Generative AI System Design Interview** (ByteByteGo, 2024) | Ali Aminian, Hao Sheng | 10 real-world GenAI design questions |
| **LLM Engineer's Handbook** (Packt, 2024) | Paul Iusztin, Maxime Labonne | Design, deploy, optimize LLMs production |

### Key Online Resources

| Resource | Type | What It Covers |
|----------|------|----------------|
| [alexeygrigorev/ai-engineering-field-guide](https://github.com/alexeygrigorev/ai-engineering-field-guide) | GitHub (★★★) | Data-driven guide: 2,445 job descriptions, 100+ interview reports, AI system design |
| [Yuan Meng - MLE Interviews 2.0](https://www.yuan-meng.com/posts/mle_interviews_2.0/) | Blog | Senior-level interview insights (2026), frontier lab interviews |
| [Chip Huyen - Building a GenAI Platform](https://huyenchip.com/2024/07/25/genai-platform.html) | Blog | Complete GenAI platform architecture by the author of DMLS |
| [InterviewNode - GenAI System Design Patterns](https://www.interviewnode.com/post/generative-ai-system-design-interview-patterns-you-should-know) | Blog | 4 repeatable patterns for GenAI interviews |
| [IGotAnOffer - GenAI System Design Interview](https://igotanoffer.com/en/advice/generative-ai-system-design-interview) | Blog | Step-by-step answer structure |
| [SystemDesignHandbook - GenAI Interview Guide](https://www.systemdesignhandbook.com/guides/generative-ai-system-design-interview/) | Blog | FAANG-focused GenAI design framework |
| [PromptLayer - Agentic System Design Interview](https://blog.promptlayer.com/the-agentic-system-design-interview-how-to-evaluate-ai-engineers/) | Blog | How companies evaluate agent design skills |
| [Bhavishya Pandit - 7 Deep-Cut AI System Design Qs](https://bhavishyapandit9.substack.com/p/7-deep-cut-ai-system-design-interview) | Substack | Real deep-cut interview questions |
| [TechEon - Complete Agentic AI System Design Guide 2026](https://atul4u.medium.com/the-complete-agentic-ai-system-design-interview-guide-2026-f95d0cfeb7cf) | Medium | 2026-specific agentic AI guide |
| [What We Learned from a Year of Building with LLMs](https://applied-llms.org/) | Blog | 6 practitioners' collective lessons, practical production insights |
| [Google Rules of Machine Learning](https://developers.google.com/machine-learning/guides/rules-of-ml) | Google Guide | Production ML best practices |

### Practice Platforms

| Platform | Focus |
|----------|-------|
| [DesignGurus](https://www.designgurus.io/) | System design courses including ML and GenAI |
| [HelloInterview](https://www.hellointerview.com/) | Company-specific interview guides (OpenAI L5 guide available) |
| [InterviewQuery](https://www.interviewquery.com/) | ML/DS interview questions and market data |
| [Exponent](https://www.tryexponent.com/) | Mock interviews with FAANG engineers |
| [Maven - AI Engineering Buildcamp](https://maven.com/alexey-grigorev/from-rag-to-agents) | 9-week intensive: RAG to Agents, production AI |

### Production Engineering Blogs (Real-World Examples)

| Company | Blog / Resource | What It Shows |
|---------|----------------|---------------|
| **Uber** | [GenAI Gateway](https://www.uber.com/blog/genai-gateway/) | Unified LLM platform, PII redactor, 60+ use cases |
| **Airbnb** | [Automation Platform v2](https://medium.com/airbnb-engineering/automation-platform-v2-improving-conversational-ai-at-airbnb-d86c9386e0cb) | LLM-powered conversational AI with CoT + guardrails |
| **Perplexity** | [ByteByteGo Analysis](https://blog.bytebytego.com/p/how-perplexity-built-an-ai-google) | 200M daily queries, RAG on Vespa.ai, Sonar models |
| **Slack** | [How We Built Slack AI](https://slack.engineering/how-we-built-slack-ai-to-be-secure-and-private/) | Stateless RAG, LLMs in escrow VPC for data privacy |
| **LinkedIn** | [QCon AI 2025](https://www.infoq.com/news/2025/12/qcon-ai-linkedin/) | AI agent platform, strict data layer siloing |
| **Anthropic** | [Multi-Agent Research](https://www.anthropic.com/engineering/multi-agent-research-system) | Opus orchestrator + Sonnet subagents, 15x more tokens |
| **DoorDash** | [Simulation Evaluation Flywheel](https://careersatdoordash.com/blog/doordash-simulation-evaluation-flywheel-to-develop-llm-chatbots-at-scale/) | AI-driven eval flywheel for LLM chatbots, hierarchical RAG |
| **Doctolib** | [Agentic AI for Healthcare](https://medium.com/doctolib/building-an-agentic-ai-system-for-healthcare-support-a-journey-into-practical-ai-implementation-0afd28d716e6) | Specialized agents in directed graph, 17K daily messages |

---

## 10. 2026 Trends & Market Data

*Source: alexeygrigorev/ai-engineering-field-guide:interview/05-trends.md (based on 130+ interview rounds reported by practitioners)*

### Job Market Numbers (2025-2026)

- **62,000 tech layoffs in 2025** — but strategic restructuring, not survival. Companies prioritize AI-fluent talent.
- **Tech job openings: ~230,000** (up 22% from 2024 lows)
- **AI-native roles surged 240%** in early 2025 (AI Engineer, ML Engineer, Analytics Engineer)
- **AI/LLM questions tripled** since 2023 in interviews
- LLM mentions in job postings: ML Engineers 8% → 29.5%, Data Scientists 4% → 18.4%, Backend Engineers 6.7% → 13%

### Interview Format Trends

| Trend | Details |
|-------|---------|
| **LeetCode declining** | ~70% of senior interviews had none. When coding appears, it's practical (real-world logic, OOP), not puzzle-based |
| **Three dominant interview types** | Coding (DSA), system design, and project presentations ("go build something in a day") |
| **AI System Design replacing "Design Dropbox"** | Scaling AI systems, latency/cost trade-offs, batching, caching, streaming, failure modes |
| **In-person finals returning** | Up from 24% (2022) to 38% (2025), driven by AI-assisted cheating concerns |
| **Project presentations common** | Present a past project, get grilled on design decisions, trade-offs, what broke, what you'd change |
| **AI-proctored early rounds** | Eightfold.ai (Jan 2026): AI agent conducts coding interview with follow-ups |

### New 2026 Interview Round Types (Senior Level)

Identified by Yuan Meng, appearing at Netflix, Snap, Reddit, DoorDash, xAI, OpenAI:

1. **ML Infra Design:** Feature stores, distributed training, online serving (not just modeling)
2. **Multi-level OOP:** Build toy backend systems with incremental complexity
3. **LLM Coding:** Implement Transformers, LoRA, KV cache, beam search from scratch
4. **Research Presentations:** Job-talk style presentations defending technical work
5. **References becoming standard:** Most top companies require 2-3 references from recent managers

### The Emerging AI Engineer Stack (2026)

```
Application:        React, Next.js, FastAPI
AI Orchestration:   LangChain, LangGraph, PydanticAI, OpenAI Agents SDK
LLM APIs:           OpenAI, Anthropic, Groq, local models (Ollama)
Vector DBs:         Pinecone, Weaviate, Qdrant, pgvector
Infrastructure:     Docker, Kubernetes, AWS/GCP/Azure
Monitoring:         Logfire, Grafana, OpenTelemetry
Evaluation:         LLM judges, Evidently, custom eval frameworks
Protocol:           Model Context Protocol (MCP) for tool integration
Multi-Agent:        A2A Protocol (Agent-to-Agent)
```

### Key Mindset Shifts

> "Can you ship, scale, and evaluate production AI?" — The bar in 2026

> "Knowledge is free — judgment isn't. Interviews shifted from 'Can you code?' to 'Can you reason with the AI that codes with you?'" — InterviewQuery 2025

> "Read the room: knowing the latest AI tools is table stakes; knowing when NOT to use them signals maturity." — Hiring manager observation, r/datascience

**The new senior bar:**
- Not "can you build an LLM app?" (everyone can)
- "Can you evaluate it properly, monitor it in production, handle failure modes, and scale it cost-effectively?"
- Junior: focuses on the prompt
- Mid: describes RAG mechanisms  
- Senior: designs an evolving ecosystem with feedback loops, versioning, monitoring, and graceful degradation

---

## Appendix: Quick Reference Cards

### Quick Reference: Interview Answer Checklist

```
□ Clarified requirements and constraints (5 min)
□ Defined success metrics: business + technical
□ Specified input/output and problem type
□ Designed data pipeline with feature engineering
□ Justified model choice with trade-offs
□ Addressed online vs batch serving
□ Included latency budget analysis
□ Described monitoring strategy (3 levels)
□ Explained retraining triggers and deployment strategy
□ Discussed failure modes and graceful degradation
□ Estimated cost and scalability
□ Mentioned A/B testing and rollout strategy
```

### Quick Reference: Classic ML vs GenAI Answer Structure

**Classic ML (Recommendation/Fraud/Search):**
Problem → Data + Features → Model → Serving → Monitoring → Retraining

**GenAI (RAG/Agent/Chatbot):**
Use Case → Context Construction (RAG) → Model + Orchestration → Guardrails → Evaluation → Feedback Loop → Cost Optimization

---

*Research compiled from:*
- *alexeygrigorev/ai-engineering-field-guide (primary source — data-driven, 2,445 real job descriptions)*
- *shafaypro/CrackingMachineLearningInterview (detailed code examples)*
- *kengio/databricks-certification-study-guide (production patterns)*
- *Yuan Meng MLE Interviews 2.0 (senior practitioner perspective)*
- *Chip Huyen GenAI Platform blog + AI Engineering book*
- *InterviewNode, SystemDesignHandbook, IGotAnOffer (interview frameworks)*
- *Engineering blogs: Uber, Airbnb, Perplexity, Slack, LinkedIn, Anthropic, DoorDash, Doctolib*
```

---

## Gaps and Uncertainties

1. **The three 2026-specific repos** (`girijesh-ai/ml-interview-codex`, `amanailab/ai-system-design-roadmap-2026`, `atryx/ai-ml-interview-2026`) appeared in GitHub search results but returned 404 — likely private repos or the slugs were inferred from search snippets and don't match actual names. The content above is sourced from verified public repos.

2. **Chatbot/Conversational AI** system architecture was covered but not from a dedicated source — content derived from alexeygrigorev's field guide and Chip Huyen's GenAI platform blog.

3. **Multi-agent protocol details** (MCP, A2A) were referenced in multiple sources but detailed architecture wasn't deeply covered — further research into `modelcontextprotocol.io` and `a2a-protocol.org` would fill this gap.

4. **Search ranking specifics** (Google Hummingbird, BERT in search) were covered at a framework level but not with real company implementation depth — the eugeneyan/applied-ml repo (which has detailed company case studies) was found but GitHub returned navigation pages rather than README content.

5. **Pricing figures** for LLM APIs are approximate (based on early 2026 estimates from sources) and may drift; the ratios (Haiku vs Sonnet vs Opus) are more reliable than absolute numbers.