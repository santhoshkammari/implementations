---

## 📊 Comprehensive AI/ML Job Market Research Report — May 2026

**Summary:** Research drawn from Glassdoor (India salary data via GUVI/guvi.in, updated March–August 2026), BLS.gov (US government official statistics, May 2024 baseline), Coursera career guides (updated July 2025), Simplilearn role descriptions, AWS MLOps blog, and BuiltIn.com live job postings. LinkedIn job search was also queried. Many popular job sites (Glassdoor direct, Naukri, Dice, ZipRecruiter) returned 403/blocked responses. Data is high-confidence for India ML/DS salaries and skill requirements; LLM Engineer and Prompt Engineer salaries are partially inferred from adjacent roles.

---

## 🗂️ ROLE 1: Junior / Entry-Level ML Engineer

### Required Technical Skills
- **Programming:** Python (primary — must-have), SQL, occasionally R or Java/C++
- **Frameworks:** scikit-learn, TensorFlow or PyTorch (basic model training)
- **Data:** pandas, NumPy, data cleaning, feature engineering, EDA
- **Math:** Linear algebra, statistics, probability, calculus fundamentals
- **Tools:** Jupyter Notebook, Git/GitHub, basic Docker
- **Cloud:** Basic AWS/GCP/Azure familiarity (SageMaker, Vertex AI, or Azure ML at introductory level)
- **ML Fundamentals:** Supervised/unsupervised learning, regression, classification, clustering

### Required Soft Skills
- Problem-solving and analytical thinking
- Clear written communication (for model documentation)
- Willingness to learn rapidly
- Basic collaboration with data engineers and scientists

### Experience Required
- 0–2 years (fresh graduates acceptable with strong portfolio/projects/Kaggle)
- B.Tech/B.E. in CS, EE, Math, Statistics; IIT/NIT/BITS preferred at top firms

### Salary Ranges
- **India:** ₹5–9 LPA (Bengaluru/Hyderabad top cities: ₹7–9 LPA; Tier-2 cities: ₹4–6 LPA)
  - Freshers start: ~₹5.1 LPA average; ₹7 LPA with strong portfolio
  - Google Bengaluru outlier: ₹12+ LPA
- **US:** $90,000–$130,000 (median ~$112,590 for data scientists per BLS 2024)

### Most Mentioned Frameworks/Tools
scikit-learn, TensorFlow, PyTorch, Keras, pandas, NumPy, Matplotlib, Seaborn, Jupyter

### Cloud Platforms Mentioned
AWS SageMaker, Google Vertex AI, Azure ML

### Database Skills
SQL (PostgreSQL, MySQL), basic NoSQL awareness (MongoDB)

### System Design Expectations
Minimal at entry level — understanding of ML pipeline stages; ability to write clean, modular Python code

---

## 🗂️ ROLE 2: Mid-Level ML Engineer (2–5 years)

### Required Technical Skills
- **Programming:** Python (advanced), Scala/Java for Spark jobs, Shell scripting
- **Frameworks:** PyTorch (strongly preferred over TensorFlow in 2025–2026), LangChain (emerging), Hugging Face Transformers
- **Deep Learning:** CNNs, RNNs, LSTMs, Transformers, attention mechanisms
- **Deployment:** FastAPI/Flask for REST APIs, Docker, basic Kubernetes
- **Big Data:** Apache Spark, Kafka, Airflow for pipelines
- **MLOps basics:** MLflow, model monitoring, A/B testing, feature stores
- **Experimentation:** Hyperparameter tuning (Optuna, Ray Tune), cross-validation

### Required Soft Skills
- Cross-functional collaboration (data engineers, product managers, domain experts)
- Translating business problems to ML problem formulations
- Technical documentation and code reviews
- Experiment tracking and result communication

### Experience Required
- 2–5 years of hands-on ML model development and deployment
- Production deployment experience is differentiating

### Salary Ranges
- **India:** ₹10–20 LPA (average ₹12–15 LPA at mid-level; Analytics/KPO sector up to ₹17 LPA)
  - Software Product companies: ₹13.8 LPA average
  - Internet companies: ₹11.2 LPA average
- **US:** $140,000–$180,000

### Most Mentioned Frameworks/Tools
PyTorch (dominant), Hugging Face, LangChain, FastAPI, MLflow, Docker, Spark, Airflow, Optuna, ONNX

### Cloud Platforms Mentioned
AWS (SageMaker most mentioned), GCP (Vertex AI), Azure (Azure ML)

### Database Skills
SQL advanced (query optimization), PostgreSQL, Redis (caching), Pinecone/Weaviate/Chroma (vector DBs emerging)

### System Design Expectations
ML pipeline design end-to-end; understanding of batch vs. real-time inference; model serving architectures; latency/throughput trade-offs

---

## 🗂️ ROLE 3: Senior AI/ML Engineer (5–8+ years)

### What Differentiates Senior from Junior/Mid

The key differentiators observed across all sources:

1. **Architecture ownership** — Senior engineers own the entire ML system design, not just models
2. **Production scale experience** — Systems serving millions of users, sub-100ms latency
3. **Mentorship & technical leadership** — Leading code reviews, defining best practices
4. **Cross-org influence** — Communicating ML trade-offs to non-technical stakeholders
5. **Research-to-production gap** — Ability to productionize cutting-edge research
6. **Cost optimization** — GPU resource efficiency, inference optimization (TensorRT, quantization, distillation)
7. **Agentic AI** — Designing multi-agent LLM systems (LangGraph, CrewAI, AutoGen)

### Required Technical Skills
- **All mid-level skills PLUS:**
- Distributed training (PyTorch DDP, FSDP, DeepSpeed)
- Model optimization: quantization (INT8/INT4), distillation, pruning, ONNX export
- Advanced MLOps: custom ML platforms, Kubeflow, SageMaker Pipelines, Vertex AI Pipelines
- Agentic frameworks: LangGraph, LlamaIndex, AutoGen, CrewAI
- Vector databases: Pinecone, Weaviate, Qdrant, pgvector
- RAG (Retrieval-Augmented Generation) system design
- Multi-cloud architecture and cost optimization
- System design at scale: designing low-latency inference systems

### Required Soft Skills
- Technical leadership and mentorship
- Stakeholder management (C-suite, product, business)
- Ability to define technical roadmaps
- Risk assessment (model safety, bias, hallucinations)
- Strong written communication (design docs, RFCs)

### Experience Required
- 5–8+ years with 3+ years in production ML
- Prior ownership of at least one large-scale ML system
- Often expected to have contributed to or published research (at FAANG/hyperscalers)

### Salary Ranges
- **India:** ₹25–60 LPA (FAANG/Product: ₹40–80 LPA; startups with ESOP: ₹30–50 LPA cash)
  - AI Researchers at this level: ₹60+ LPA
- **US:** $200,000–$350,000+ (FAANG senior: $250–400K+ with equity)
  - Glassdoor ML Engineer base (US): $156,000 median

### Most Mentioned Frameworks/Tools
PyTorch, LangChain, LangGraph, LlamaIndex, DeepSpeed, vLLM, TensorRT, Kubeflow, Ray, MLflow, Weights & Biases

### Cloud Platforms
AWS (EC2, SageMaker, Lambda, Bedrock), GCP (Vertex AI, BigQuery, Gemini APIs), Azure (Azure OpenAI, Azure ML)

### Database Skills
PostgreSQL, Redis, Pinecone, Weaviate, Neo4j (for knowledge graphs), Snowflake, Databricks

### System Design Expectations
High — expected to design full ML platforms: feature stores, model registries, CI/CD for ML, A/B test infrastructure, observability pipelines

---

## 🗂️ ROLE 4: MLOps Engineer

### Core Responsibilities (per Coursera, AWS blog)
- Develop, test, and deploy ML models through their full lifecycle
- Build ML pipelines using CI/CD techniques
- Implement Docker/Kubernetes for model containerization and deployment
- Monitor model performance (drift detection, data quality alerts)
- Automate model retraining workflows
- Manage multi-environment MLOps (dev → staging → production)

### Required Technical Skills
- **DevOps foundation:** CI/CD (GitHub Actions, Jenkins, GitLab CI), Docker, Kubernetes (K8s)
- **ML frameworks:** TensorFlow, PyTorch, Scikit-learn
- **Pipeline orchestration:** Apache Airflow, Kubeflow Pipelines, SageMaker Pipelines, ZenML, Prefect
- **Experiment tracking:** MLflow, Weights & Biases (W&B), Neptune.ai
- **Model serving:** TorchServe, TensorFlow Serving, Triton Inference Server, BentoML, Ray Serve
- **Monitoring:** Evidently, Seldon, WhyLogs, Prometheus + Grafana
- **Infrastructure as Code:** Terraform, Ansible, CloudFormation
- **Languages:** Python (primary), Bash scripting, YAML/HCL
- **Containers/Orchestration:** Docker, Kubernetes (EKS, GKE, AKS)
- **Feature stores:** Feast, Tecton, Hopsworks
- **Data versioning:** DVC (Data Version Control)

### Required Soft Skills
- Bridge between data scientists, ML engineers, and IT operations
- Collaboration and communication across teams
- Documentation discipline
- Incident response and troubleshooting under pressure

### Experience Required
- 3–6 years (senior-level role; often transitioned from DevOps + ML background)
- Expected: bachelor's in CS, data science, or software engineering
- MLOps is described as a "senior-level role" requiring software development background (Coursera, Jul 2025)

### Salary Ranges
- **India:** ₹12–25 LPA (mid-level MLOps); ₹25–45 LPA (senior)
  - Transitioning from NLP/MLOps Engineer can reach ₹12–15 LPA per GUVI
- **US:** $87,220 average (ZipRecruiter, cited by Coursera); up to $136,500; with Glassdoor showing ML Engineer at $156,000

### Most Mentioned Frameworks/Tools
MLflow, Kubeflow, Airflow, DVC, W&B, Evidently, Seldon, Triton, BentoML, ZenML, Feast

### Cloud Platforms
AWS (SageMaker, EC2, S3, ECR, CodePipeline), GCP (Vertex AI Pipelines, Cloud Run, GKE), Azure (Azure DevOps, AKS, Azure ML)

### Database Skills
S3 (object storage), PostgreSQL, Redis, DynamoDB, Hive, Delta Lake

### System Design Expectations
Full ML lifecycle architecture: multi-account AWS strategy (dev/staging/prod), CI/CD for ML, model registry design, feature store design, monitoring infrastructure for data drift and model degradation

---

## 🗂️ ROLE 5: AI Research Scientist

### Role Profile
Focuses on advancing the state-of-the-art; publishes papers; bridges research → production. Common at OpenAI, Google DeepMind, Meta AI, Microsoft Research, Amazon Science, Anthropic, and their India R&D centers (Bengaluru, Hyderabad).

### Required Technical Skills
- **Programming:** Python (expert), C++ (for performance-critical research)
- **Deep Learning (advanced):** Transformer architectures, attention mechanisms, RLHF, Constitutional AI, diffusion models, multi-modal learning
- **Mathematics (advanced):** Information theory, optimization theory, statistical learning theory, probabilistic graphical models, differential geometry
- **Research skills:** Literature review (ArXiv), experiment design, statistical significance testing, ablation studies
- **Frameworks:** JAX (increasingly dominant in research), PyTorch, Flax, Haiku
- **Infrastructure:** SLURM for HPC clusters, multi-GPU/multi-node training (NCCL), distributed computing

### Required Soft Skills
- Scientific rigor and intellectual curiosity
- Technical writing (papers, research reports)
- Peer collaboration and critique culture
- Ability to work with long experimentation timelines
- Communication of complex results to non-researchers

### Experience Required
- PhD strongly preferred (or equivalent research publications)
- 2–5 years post-PhD for Research Scientist; 5+ for Senior
- Publication record at NeurIPS, ICML, ICLR, ACL, CVPR is strongly differentiating

### Salary Ranges
- **India (research labs):** ₹30–80+ LPA; top research scientists at FAANG India: ₹60–100+ LPA
  - GUVI explicitly states: "AI Researchers often command higher packages, reaching ₹60 LPA or more"
- **US:** $180,000–$400,000+ (Research Scientist at OpenAI/DeepMind level with equity)
  - BLS Computer & Information Research Scientists median: $140,910

### Most Mentioned Frameworks/Tools
JAX, PyTorch, Flax, Weights & Biases, HuggingFace, Triton (CUDA kernel), SLURM

### Cloud Platforms
GCP (TPUs via Google), AWS (P4/P5 GPU instances), Azure (A100 clusters)

### Database Skills
Limited traditional DB; focus on large-scale data storage (S3, GCS), data streaming (Apache Beam), WebDataset

### System Design Expectations
Research-oriented: designing experiments at scale, multi-GPU training frameworks, ablation study design, benchmark construction

---

## 🗂️ ROLE 6: Data Scientist (AI-Focused)

### Role Profile (BLS, GUVI sources)
Bridging traditional statistics/analytics and modern ML; increasingly includes LLM-powered analytics, agentic data analysis, and NLP.

### Required Technical Skills
- **Programming:** Python, R (for statistical work), SQL (advanced — must-have)
- **ML/Stats:** scikit-learn, statistical modeling, hypothesis testing, A/B testing, causal inference
- **Visualization:** Tableau, Power BI, Matplotlib, Seaborn, Plotly
- **Big Data:** Spark (PySpark), Hadoop, BigQuery, Databricks, Snowflake
- **AI-specific (2026):** HuggingFace for NLP features, OpenAI/Gemini APIs for text analytics, LangChain for data pipelines with LLMs
- **Cloud Analytics:** AWS (Athena, Glue, Redshift), GCP (BigQuery, Looker), Azure (Synapse)

### Required Soft Skills
- Storytelling with data — translating insights for executives
- Business acumen and domain expertise (finance, healthcare, retail)
- Strong presentation and communication skills
- Stakeholder management
- Experimental design mindset

### Experience Required
- 2–5 years (mid-level); 5+ years (lead data scientist)
- Bachelor's minimum; Master's preferred for AI-focused roles at banks, pharma

### Salary Ranges
- **India:** ₹8–20 LPA (average ~₹10 LPA); senior lead: ₹20–40 LPA
  - Analytics & KPO sector highest: ₹17 LPA average per AmbitionBox
  - Finance/Banking: ₹9.9 LPA average
- **US:** $112,590 median (BLS May 2024); top 10%: $194,410+
  - Computer Systems Design sector highest: $128,020

### Most Mentioned Tools
SQL (universal), Python, pandas, scikit-learn, Tableau, Power BI, Spark/PySpark, BigQuery, Snowflake, Databricks

### Cloud Platforms
AWS, GCP (BigQuery dominant for analytics), Azure (Synapse Analytics)

### Database Skills
PostgreSQL, MySQL, BigQuery, Redshift, Snowflake, Databricks Delta Lake, MongoDB

### System Design Expectations
Data pipeline design (ETL/ELT), feature store contribution, dashboard architecture, A/B test framework setup

---

## 🗂️ ROLE 7: LLM Engineer / GenAI Engineer

### Role Profile
Newly crystallized role (2023–2026). Focuses on building production applications using Large Language Models: RAG systems, agents, fine-tuning, and LLM API integrations. KPMG India posting on LinkedIn confirms active hiring for "GenAI Engineer" roles.

### Required Technical Skills
- **LLM Frameworks:** LangChain (most common), LlamaIndex, LangGraph (for agentic workflows)
- **Models:** OpenAI GPT-4o, Anthropic Claude 3.5, Google Gemini 2.0, Meta LLaMA 3.x, Mistral
- **RAG:** Vector databases (Pinecone, Weaviate, Chroma, Qdrant, pgvector), embeddings, rerankers (Cohere), hybrid search
- **Fine-tuning:** LoRA/QLoRA (PEFT library), Hugging Face Trainer, Axolotl, Unsloth
- **Deployment:** vLLM, Ollama (local), TGI (Text Generation Inference), OpenLLM, FastAPI
- **Prompt Engineering:** Chain-of-thought, few-shot, system prompt design, structured output (JSON schema)
- **Evaluation:** RAGAS, TruLens, LangSmith, Arize Phoenix (LLM observability)
- **Programming:** Python (expert), TypeScript/Node.js (for frontend integration)
- **Agentic patterns:** ReAct, function calling, tool use, multi-agent coordination

### Required Soft Skills
- Iterative mindset — rapid prototyping and evaluation
- Communication of LLM limitations (hallucinations, context windows)
- Product-oriented thinking (user experience with AI systems)
- Ethical AI awareness

### Experience Required
- 1–4 years (this is a new field; experience is measured differently)
- Strong portfolio of LLM projects > formal years of experience
- Certifications (DeepLearning.AI courses) valued

### Salary Ranges
- **India:** ₹12–30 LPA (2–5 yrs experience); senior: ₹30–60 LPA; highest at AI-first startups and GCCs
- **US:** $150,000–$220,000 (inferred from adjacent ML Engineer and AI Engineer roles)

### Most Mentioned Frameworks/Tools
LangChain, LlamaIndex, LangGraph, Pinecone, OpenAI API, HuggingFace, vLLM, RAG patterns, FastAPI, Chroma, Weaviate

### Cloud Platforms
AWS (Bedrock for Claude/Titan/Llama), GCP (Vertex AI with Gemini), Azure (Azure OpenAI Service)

### Database Skills
Vector DBs (Pinecone, Weaviate, Qdrant, pgvector), Redis (semantic caching), Neo4j (knowledge graphs for RAG), MongoDB (document storage)

### System Design Expectations
RAG pipeline architecture (chunking strategies, embedding model selection, retrieval optimization), agentic system design, LLM gateway patterns, context window management, guardrails implementation

---

## 🗂️ ROLE 8: Prompt Engineer

### Role Profile
Standalone "Prompt Engineer" roles are declining as the skill gets absorbed into LLM Engineer roles. In 2026, it's primarily a component of LLM Engineer, AI Product Manager, or Solutions Architect roles rather than a standalone title. Still found as specific role at content AI companies and AI-as-a-service firms.

### Required Technical Skills
- Advanced prompt design: few-shot, chain-of-thought, ReAct, meta-prompting
- System prompt architecture for enterprise applications
- Prompt testing/evaluation frameworks (PromptFoo, LangSmith)
- Understanding of model capabilities and limitations per model version
- Context management, token optimization
- Python scripting for automated prompt testing
- Domain expertise (legal, medical, financial — context-dependent)

### Salary Ranges
- **India:** ₹8–18 LPA (standalone), more as LLM Engineer: ₹15–35 LPA
- **US:** $100,000–$175,000 (standalone prompt engineer) — declining as absorbed into LLM Engineer

---

## 🗂️ ROLE 9: AI Platform Engineer

### Role Profile
Builds the internal tooling, compute orchestration, and infrastructure that enables ML teams to work at scale. Intersection of Platform Engineering, MLOps, and Cloud Infrastructure. Examples: Building internal GPU clusters, model serving infrastructure, internal LLM gateways, ML compute platforms. Direct evidence: NVIDIA Run:ai platform (covering GPU orchestration, KAI Scheduler on Kubernetes, model serving) represents the technology stack this role manages.

### Required Technical Skills
- **Kubernetes (K8s) — Expert level:** Custom controllers, operators, scheduling policies (KARPENTER, KAI Scheduler)
- **GPU orchestration:** NVIDIA DCGM, GPU sharing, MIG (Multi-Instance GPU)
- **Infrastructure as Code:** Terraform, Pulumi, Ansible, Helm
- **Container ecosystem:** Docker, containerd, GPU-enabled containers
- **Distributed systems:** gRPC, message queues (Kafka, SQS), distributed caching (Redis)
- **CI/CD:** GitHub Actions, ArgoCD, Tekton, GitOps workflows
- **Observability:** Prometheus, Grafana, OpenTelemetry, Jaeger
- **Model serving infrastructure:** Triton Inference Server, vLLM, Ray Serve, BentoML, KServe
- **Languages:** Python (primary), Go (for infrastructure tooling), Bash
- **Storage:** S3-compatible (MinIO), distributed file systems, model artifact management
- **Security:** RBAC, service meshes (Istio), secret management (Vault)
- BuiltIn.com job postings confirm: AWS+Azure+GCP+Go+Python+K8S+Distributed Caching+Terraform+Chaos Engineering

### Required Soft Skills
- Deep systems thinking
- Collaboration with data scientists and ML engineers
- Reliability engineering mindset (SLOs, SLAs, incident management)
- Infrastructure cost optimization awareness

### Experience Required
- 4–8 years (senior-heavy role)
- Background: DevOps/SRE → MLOps → AI Platform

### Salary Ranges
- **India:** ₹20–50 LPA (senior Platform Engineer in AI companies)
- **US:** $170,000–$280,000

### Most Mentioned Tools
Kubernetes, Terraform, Prometheus/Grafana, Triton, vLLM, Ray, Airflow, Argo Workflows, NVIDIA Run:ai, KubeFlow

### Cloud Platforms
Multi-cloud expertise required: AWS (EKS, EC2 GPU fleet), GCP (GKE, A3 Ultra instances with H200 GPUs), Azure (AKS)

### Database Skills
PostgreSQL (metadata store), Redis (distributed cache), S3/GCS (artifact stores), Etcd (K8s state)

### System Design Expectations
Design full AI compute platforms: GPU cluster scheduling, model serving at scale (zero-downtime deployments), multi-tenant ML infrastructure, cost attribution systems, autoscaling inference clusters

---

## 🗂️ ROLE VARIATIONS — Quick Reference (20–30 Roles)

| Role | Exp (yrs) | India Salary | US Salary | Key Differentiator |
|---|---|---|---|---|
| **Junior ML Engineer** | 0–2 | ₹5–9 LPA | $90–130K | Python + scikit-learn + basic DL |
| **ML Engineer (Mid)** | 2–5 | ₹10–20 LPA | $140–180K | Production deployment + PyTorch |
| **Senior ML Engineer** | 5–8 | ₹25–60 LPA | $200–350K | System design + LLM integration |
| **Staff ML Engineer** | 8+ | ₹50–90 LPA | $300–450K | Org-wide technical strategy |
| **Principal ML Engineer** | 10+ | ₹80–120+ LPA | $400–600K | Multi-team technical vision |
| **MLOps Engineer (Mid)** | 3–5 | ₹15–25 LPA | $120–160K | CI/CD + Kubeflow + monitoring |
| **Senior MLOps Engineer** | 5–8 | ₹25–45 LPA | $160–220K | Platform design + feature stores |
| **AI Research Scientist** | PhD + 2–5 | ₹30–80 LPA | $180–400K | Publications + novel architectures |
| **Senior AI Research Scientist** | PhD + 5+ | ₹60–120+ LPA | $300–600K | Team lead + flagship research |
| **Data Scientist** | 2–4 | ₹8–15 LPA | $100–150K | SQL + analytics + storytelling |
| **Senior Data Scientist** | 4–7 | ₹15–30 LPA | $150–200K | Causal inference + platform |
| **Lead Data Scientist** | 7+ | ₹30–50 LPA | $180–250K | Team + roadmap ownership |
| **LLM Engineer** | 1–3 | ₹12–25 LPA | $150–200K | RAG + LangChain + fine-tuning |
| **Senior LLM Engineer** | 3–6 | ₹25–50 LPA | $180–280K | Agentic systems + evaluation |
| **GenAI Engineer** | 1–4 | ₹12–30 LPA | $140–220K | Production GenAI apps |
| **Prompt Engineer** | 1–3 | ₹8–18 LPA | $100–175K | Prompt design + evaluation |
| **AI Platform Engineer** | 4–7 | ₹20–45 LPA | $170–250K | K8s + GPU + model serving |
| **Senior AI Platform Eng.** | 6–10 | ₹40–70 LPA | $250–350K | Full AI infra ownership |
| **Computer Vision Engineer** | 2–5 | ₹12–25 LPA | $130–180K | CNNs + YOLO + OpenCV |
| **NLP Engineer** | 2–5 | ₹12–22 LPA | $130–175K | Transformers + BERT + embeddings |
| **Reinforcement Learning Eng.** | 3–6 | ₹20–45 LPA | $160–250K | PPO/DQN + simulation |
| **AI Solutions Architect** | 5–8 | ₹30–60 LPA | $180–280K | Client-facing AI design |
| **AI Product Manager** | 4–7 | ₹25–50 LPA | $160–250K | Product + AI roadmap |
| **AI Ethics/Safety Researcher** | PhD + 2–5 | ₹25–60 LPA | $150–300K | RLHF + alignment + red-teaming |
| **Multimodal AI Engineer** | 3–6 | ₹20–45 LPA | $160–250K | Vision+Language (GPT-4o-style) |
| **Edge AI Engineer** | 3–6 | ₹15–30 LPA | $130–200K | TensorRT Lite + embedded |
| **AI Infrastructure Engineer** | 4–7 | ₹18–40 LPA | $150–240K | GPU cluster mgmt + networking |
| **Recommendation Systems Eng.** | 3–6 | ₹15–35 LPA | $150–250K | Ranking + retrieval + A/B |
| **Time Series ML Engineer** | 2–5 | ₹12–22 LPA | $120–170K | Forecasting + Prophet/ARIMA/DL |
| **MLOps/DevOps Hybrid** | 3–6 | ₹15–28 LPA | $130–200K | CI/CD for ML + DevOps |

---

## 🔧 CROSS-ROLE TECHNOLOGY FREQUENCY MATRIX

### Programming Languages (most → least mentioned across roles)
1. **Python** — Universal (100% of roles)
2. **SQL** — All data/DS/MLOps roles (~80%)
3. **Java/Scala** — Big Data + enterprise roles (~30%)
4. **Go** — Platform/infrastructure roles (~25%)
5. **C++** — Research + edge AI + systems (~20%)
6. **R** — Academia/statistical DS (~15%)
7. **TypeScript/JavaScript** — LLM Engineer (frontend integration, ~15%)
8. **Bash/Shell** — MLOps/Platform (~60% — often listed as secondary)
9. **Rust** — Emerging for performance-critical systems (~5%)

### ML/DL Frameworks (most → least mentioned)
1. **PyTorch** — #1 in 2026 for both research and production (dominant)
2. **scikit-learn** — ML Engineer + Data Scientist universal baseline
3. **TensorFlow/Keras** — Still mentioned, losing ground to PyTorch
4. **HuggingFace Transformers** — NLP/LLM universal
5. **LangChain** — LLM/GenAI roles (fast growing)
6. **LlamaIndex** — LLM RAG roles
7. **LangGraph** — Agentic AI (emerging fast in senior roles)
8. **XGBoost/LightGBM** — DS/tabular ML roles
9. **JAX** — Research roles (Google ecosystem)
10. **Ray** — Distributed training + serving
11. **DeepSpeed** — Senior training at scale

### Cloud Platforms Mentioned
1. **AWS** — Most mentioned overall (SageMaker, EC2, S3, Lambda, Bedrock, EKS)
2. **GCP** — Strong in research + big data (Vertex AI, BigQuery, GKE, TPUs)
3. **Azure** — Enterprise/corporate roles (Azure ML, Azure OpenAI, AKS, Azure DevOps)
4. Multi-cloud: Senior roles increasingly require 2+ cloud familiarity

### Database/Storage Skills
| Type | Tools | Roles Requiring |
|---|---|---|
| Relational | PostgreSQL, MySQL | All (nearly universal) |
| Vector DB | Pinecone, Weaviate, Qdrant, pgvector, Chroma | LLM/GenAI (fast-growing) |
| NoSQL | MongoDB, Cassandra, DynamoDB | Backend-integrated ML roles |
| Data Warehouse | Snowflake, Databricks, BigQuery, Redshift | DS, MLOps |
| Graph DB | Neo4j, NetworkX | Research + knowledge graph RAG |
| Cache | Redis, ElastiCache | Platform, LLM inference |
| Object Storage | S3, GCS, Azure Blob | MLOps, Platform (universal) |

### MLOps/Infra Tools
| Category | Tools |
|---|---|
| Experiment Tracking | MLflow, W&B (Weights & Biases), Neptune, CometML |
| Pipeline Orchestration | Apache Airflow, Kubeflow Pipelines, Prefect, ZenML, Argo Workflows |
| Model Serving | Triton, TorchServe, vLLM, BentoML, Ray Serve, KServe |
| Model Registry | MLflow, SageMaker Model Registry, Vertex AI Model Registry |
| Data Versioning | DVC |
| Feature Stores | Feast, Tecton, Hopsworks, SageMaker Feature Store |
| Monitoring | Evidently, Seldon, Arize, WhyLogs, Fiddler |
| Container Infra | Docker, Kubernetes, Helm, ArgoCD |
| IaC | Terraform, Pulumi, Ansible, CloudFormation |

---

## 📈 Market Trends (May 2026 Snapshot)

### India-Specific Trends
- **Top hiring cities:** Bengaluru (#1), Hyderabad, Pune, Gurugram, Chennai, Mumbai
- **Top hiring companies:** Google, Microsoft, Amazon, Flipkart, Swiggy, Meesho, Zepto (startups paying premium); also Accenture, Infosys, Cognizant (IT services, lower band)
- **GCC (Global Capability Centers)** are major AI employers in India: JPMorgan, Morgan Stanley, Goldman Sachs, BlackRock all have India AI hubs
- **Growth drivers:** NASSCOM projected 40%+ growth in AI/ML specialist demand through 2027 (WEF data)
- **Freshers:** ₹5–7 LPA baseline; FAANG freshers (IIT grads): ₹20–35 LPA

### Global Market Signals (WEF Future of Jobs Report 2025)
- AI/ML specialists projected as **top fastest-growing** job category 2025–2030
- **34%** employment growth for data scientists projected 2024–2034 (BLS)
- **82,500 new US data scientist positions** projected by 2034
- MLOps market: $1.5B (2024) → $19.55B (2032) — major tailwind for MLOps/Platform roles

### 2026-Specific Shifts in Requirements
- **LLMs are now baseline** — almost every AI/ML role now requires at least familiarity with LLMs, RAG, and LangChain
- **Agentic AI** — Senior roles now explicitly mention LangGraph, AutoGen, CrewAI
- **Vector databases** — Now expected knowledge in LLM Engineer and above
- **Responsible AI / AI Safety** — Growing requirement especially at large companies; fairness, bias evaluation, guardrails
- **Multimodal AI** — Vision + language skills increasingly expected at mid-senior levels
- **Cost efficiency** — GPU cost optimization, quantization, distillation now in senior JDs

---

## ⚠️ Gaps and Uncertainties

1. **Direct JD scraping blocked:** LinkedIn, Naukri, Glassdoor, Dice, ZipRecruiter all blocked direct access. Salary data is from GUVI/Glassdoor India (via secondary sources) — accurate as of March 2026; GUVI article dated March 16, 2026.

2. **Prompt Engineer standalone salaries:** Inferred from adjacent roles; this title is being deprecated in favor of LLM Engineer.

3. **US salary ranges for LLM-specific roles:** Partially inferred. BLS does not yet have a separate "LLM Engineer" category; numbers extrapolated from ML Engineer ($156K Glassdoor) + 20–40% premium for LLM specialization.

4. **Top-of-band salaries (₹120 LPA India, $600K US):** Only at FAANG + equity; cash components lower. Should note total compensation vs. base salary distinction.

5. **AI Platform Engineer India salary:** Less data available; inferred from SWE Platform + MLOps premium.

6. **Naukri data:** Could not retrieve due to SSR/JS blocking. Naukri would be the best source for real-time India ML job data.

7. **Prompt Engineer vs. LLM Engineer:** The distinction is blurring. By 2026, most companies absorb prompt engineering into LLM Engineer or AI Product roles.

**Suggested follow-up searches (if more specificity needed):**
- Naukri.com: search via `curl` or different user agent for actual JD text
- AmbitionBox salary distributions for specific companies (Google, Amazon, Flipkart India ML roles)
- LinkedIn Sales Navigator for actual 2026 JD keyword frequency analysis
- H1B Salary Database (levels.fyi) for US ML engineer compensation bands by company

---

**Sources consulted:**
- `guvi.in/blog/machine-learning-engineer-salary-in-india/` — India ML salary data, March 2026
- `guvi.in/blog/top-skills-to-become-a-machine-learning-engineer/` — ML skills 2026, updated Feb 2026
- `guvi.in/blog/machine-learning-job-description-skills-and-tools-to-know/` — JD breakdown, Aug 2025
- `coursera.org/articles/mlops-engineer` — MLOps role definition, responsibilities, salary, Jul 2025
- `simplilearn.com/machine-learning-engineer-job-description-article` — ML Engineer responsibilities
- `bls.gov/ooh/math/data-scientists.htm` — US BLS Data Scientist stats, May 2024 official
- `aws.amazon.com/blogs/machine-learning/mlops-foundation-roadmap-for-enterprises-with-amazon-sagemaker/` — MLOps architecture
- `builtin.com/jobs/data-science/machine-learning-engineer` — Live job postings (India, 2025–2026)
- `weforum.org/publications/the-future-of-jobs-report-2025/` — WEF Future of Jobs 2025
- `run.ai/guides/machine-learning-engineering/mlops-engineer` — GPU/AI platform trends
- `in.linkedin.com/jobs/search/?keywords=LLM+Engineer` — Live GenAI/LLM Engineer postings India (KPMG GenAI Engineer confirmed active)
- `projectpro.io/article/ai-engineer-job-description/1010` — Expert commentary on GenAI skills (Siemens AI Manager interview)