# EduSight Africa — AI-Ready Deployment and Scaling Plan

> Proposed future-state architecture, not a statement of what is deployed. Keep Hugging Face central to model management while supporting cost-efficient Qwen and GLM inference through interchangeable providers.

## 1. Revised architecture

The following diagram reproduces the previously agreed components and connections in GitHub-renderable Mermaid.

```mermaid
flowchart TB
    U["EduSight Africa users<br/>Teachers · Parents · School Administrators"]
    F["Next.js — Vercel<br/>Web interface · Authentication · Analytics dashboards"]
    B["FastAPI — Render<br/>REST APIs · Authorization · Business logic · Assessment workflows"]
    subgraph AI["AI Orchestration Layer"]
      O["Model routing · Prompt management · Fallbacks · Evaluation · Observability"]
      M["Classical ML<br/>XGBoost · Random Forest · Risk predictions"]
      L["LLM Gateway<br/>Qwen · GLM · Provider switching"]
      H["Hugging Face<br/>Model registry · Spaces · Hosted inference"]
      P["Other inference providers<br/>Cost-optimized Qwen / GLM APIs"]
      O --> M
      O --> L
      M --> H
      L --> H
      L --> P
    end
    D[("Supabase<br/>PostgreSQL · pgvector")]
    R[("Redis<br/>Caching · Rate limiting")]
    U --> F --> B --> O
    B --> D
    B -. optional .-> R
```

Start with AI orchestration as a Python package inside FastAPI; extract it into a microservice when traffic or deployment requirements justify it. Hugging Face is the model hub and an optional inference layer, **not necessarily the runtime for every LLM request**.

## 2. Hosting strategy

| Component | Platform | Strategy |
|---|---|---|
| Frontend | Vercel | Free non-commercial development |
| Backend | Render | Free initially; upgrade for reliability |
| PostgreSQL | Supabase | Free tier initially |
| Classical ML serving | Hugging Face Spaces | CPU Basic public prototype, if account/plan permits; portable container elsewhere otherwise |
| ML model registry | Hugging Face Hub | Versioned model artifacts |
| LLM inference | Hugging Face Inference Providers / Qwen or GLM APIs | Free credits or usage-based pricing |
| Vector database | Supabase pgvector | Add when RAG is needed |
| Redis | Optional hosted Redis | Add when caching or queues justify it |
| Model monitoring | Custom logs initially | Track latency, cost, failures, model versions |

**Important limitations:** Public Spaces must only receive synthetic or otherwise safe demonstration data, not identifiable student records. Free CPU Spaces can sleep; compute-Space creation eligibility and paid inference pricing must be rechecked before provisioning. Do not rely on free inference credits for production workloads. Render free instances can cold-start; plan for timeouts and eventual paid capacity.

## 3. Qwen and GLM routing

| Future capability | Preferred model family | Purpose |
|---|---|---|
| Student assessment summaries | Qwen small instruct model | Explain risk indicators in accessible language |
| Teacher AI assistant | Qwen | Answer questions about authorized student records |
| Educational intervention plans | Qwen or GLM | Generate draft support recommendations |
| Complex analysis | GLM | Multi-step reasoning and planning |
| Multilingual communication | Qwen multilingual model | English, Kiswahili and other supported languages |
| RAG over school policies | Qwen + embedding model | Ground responses in school documents |
| Agentic workflows | GLM or Qwen tool-calling model | Query approved APIs and execute authorized actions |

Benchmark Qwen and GLM on the same task set for grounding, Kiswahili quality, latency, safety, and cost. LLMs should **explain or contextualize validated risk predictions**, not independently determine a student's risk classification.

## 4. Target repository structure

```text
edusight-africa/
├── frontend/
├── backend/
│   └── app/
│       ├── api/
│       ├── services/
│       ├── ml/
│       └── ai/
│           ├── gateway.py
│           ├── providers/
│           │   ├── huggingface.py
│           │   ├── qwen.py
│           │   └── glm.py
│           ├── routing.py
│           ├── prompts/
│           ├── guardrails.py
│           └── evaluation/
├── ml/
│   ├── features.py
│   ├── train_model.py
│   └── models/
├── inference/
│   ├── Dockerfile
│   ├── app.py
│   └── requirements.txt
├── docs/
│   └── deployment/
└── docker-compose.yml
```

This is a **proposed** structure, not an assertion that these paths currently exist. The independent `inference/` service should initially serve classical ML, be Docker-portable, and optionally run on Hugging Face Spaces.

Example configuration (set actual secrets through deployment secret managers):

```dotenv
AI_PROVIDER=huggingface
AI_MODEL=your-selected-model-id
HF_TOKEN=your-huggingface-token

AI_FALLBACK_PROVIDER=qwen
AI_REQUEST_TIMEOUT_SECONDS=30
AI_MAX_OUTPUT_TOKENS=512
```

Implement request quotas per school, token/cost logging, retry/backoff limits, provider fallbacks, and AI feature flags.

## 5. Future RAG architecture

```mermaid
flowchart LR
    DOC["Authorized school policies and documents"] --> ING["Ingestion<br/>Parse · Chunk · Metadata"]
    ING --> EMB["Embedding model"]
    EMB --> V[("Supabase pgvector<br/>Tenant-scoped vectors")]
    Q["Authenticated teacher question"] --> AUTH["Authorization and school scope"]
    AUTH --> RET["Filtered retrieval<br/>Citations"]
    V --> RET
    RET --> PROMPT["Grounded prompt"]
    PROMPT --> G["LLM Gateway<br/>Qwen / GLM"]
    G --> OUT["Cited answer<br/>with uncertainty"]
```

Enforce tenant isolation at both the database and retrieval layers; evaluate retrieval recall, faithfulness, refusal behavior, and multilingual performance.

## 6. Future agentic architecture

```mermaid
flowchart LR
    T["Authorized teacher"] --> API["FastAPI auth + RBAC"]
    API --> A["Agent orchestrator<br/>Qwen / GLM tool calling"]
    A --> P["Tool policy<br/>Approval · Audit · Quotas"]
    P --> S["Read authorized student data"]
    P --> I["Draft intervention plan"]
    P --> R["Create teacher review task<br/>with approval"]
    S --> A
    I --> A
    R --> A
    A --> T
```

Tool calls must be allowlisted, tenant-scoped, logged, and subject to human approval for consequential actions.

## 7. Scaling roadmap

### Phase 1 — Free demonstration deployment
- Deploy Next.js, FastAPI, and PostgreSQL; publish versioned ML artifacts to Hugging Face Hub.
- Build an isolated, portable CPU inference service; use synthetic data in public demos.
- Start with rule-based inference until real trained artifacts and compatible dependencies are packaged.
- Run Alembic migrations; configure production CORS, authentication, secrets, and model readiness reporting.
- Validate end-to-end behavior and cold-start latency.

### Phase 2 — LLM integration
- Add provider-neutral gateway and Hugging Face, Qwen, and GLM adapters.
- Use Qwen for grounded assessment explanations; benchmark GLM for fallback and complex tasks.
- Track model version, quality, latency, token usage, provider costs, and failure rates.

### Phase 3 — RAG and educational intelligence
- Add pgvector, document ingestion, versioning, embeddings, and tenant-scoped retrieval.
- Produce grounded, cited responses and evaluate hallucination and access-control risks.

### Phase 4 — Agentic workflows and scaling
- Add approved tools for attendance lookup, intervention drafts, and teacher review tasks.
- Introduce queues, background workers, caching, dedicated inference, monitoring, and paid hosting when needed.
- Require human review before consequential changes to student records.

### Deployment evolution

```mermaid
flowchart TB
    subgraph ONE["Phase 1 · Demo"]
      F1["Vercel frontend"] --> B1["Render FastAPI"]
      B1 --> DB1[("Supabase Postgres")]
      B1 --> ML1["Rule-based / CPU ML inference"]
      ML1 -. model versions .-> HF1["Hugging Face Hub"]
    end
    subgraph TWO["Phase 2 · AI-assisted"]
      B2["FastAPI + AI gateway"] --> ML2["Classical ML"]
      B2 --> L2["Qwen / GLM provider APIs"]
      L2 -. model discovery .-> HF2["Hugging Face ecosystem"]
    end
    subgraph THREE["Phase 3 · Knowledge grounded"]
      B3["Authorized FastAPI"] --> RET3["RAG retrieval"]
      RET3 --> DB3[("Supabase pgvector")]
      RET3 --> L3["Qwen / GLM"]
    end
    subgraph FOUR["Phase 4 · Production scale"]
      E4["Frontend + API"] --> J4["Queues + workers"]
      J4 --> I4["Dedicated inference"]
      J4 --> A4["Audited agent tools"]
      E4 --> O4["Monitoring + cost controls"]
    end
    ONE --> TWO --> THREE --> FOUR
```

## 8. Risks and acceptance criteria

1. **Model packaging:** Current model metadata refers to local paths; ship real artifacts, pin compatible Python ML dependencies, and verify active serving mode.
2. **Prediction quality:** Existing evaluation showed poor identification of the critical-risk class; do not use for consequential student decisions until independently validated on representative data and reviewed by educators.
3. **Privacy:** Protect children's records through least privilege, retention limits, appropriate legal basis, vendor review, and no identifiable data in public Spaces.
4. **Cost:** Set per-tenant budgets, alerts, token limits, and explicit fallback behavior. Recheck hosting/inference terms before deployment.
5. **Resilience:** LLM provider outages must not disable core assessment workflows.
6. **Operations:** Test migrations, CORS, authentication, cold starts, observability, rollback, and tenant isolation.

**Decision:** Build EduSight Africa as a model-agnostic AI platform, with Hugging Face for model management and optional serving, Qwen and GLM for cost-aware inference, and independently deployable web, data, and AI layers.
