# SD-05: Interview Case Studies (Full Solved Designs)

> Yeh sabse important file — actual interview questions ke complete walkthroughs. SD-01 se SD-04 ka knowledge yahan apply hota. Har case study framework (SD-00) follow karta. Yeh padhke tu confidently whiteboard pe design kar payega.

---

## Kaise Use Karna

Har case study me main framework follow karunga: requirements → scale → high-level design → deep dive → bottlenecks/scale → trade-offs → failure. Tu ise padhke **pattern seekhega** — phir koi bhi naya question aaye, same framework apply karega. Interview me yeh bolna: "Let me clarify requirements first" — phir structured aage badhna. Yeh discipline hi seniority dikhata.

---

## CASE STUDY 1: Design an LLM Serving Platform (Most Common)

**Question:** "Design a platform to serve an LLM (like a chatbot) to 100k users, 1000 requests/sec, low latency."

**Requirements clarify:** Functional — user query bheje, LLM response mile, streaming chahiye (perceived latency). Non-functional — 1000 QPS, p99 latency budget (say 2-3 sec acceptable for LLM), 99.9% availability, cost-conscious. Model size? (say 13B, needs GPU). Multi-region? (say single region for now).

**Scale estimate:** 1000 QPS, har request LLM inference. Ek GPU (A10/A100) vLLM ke saath maybe 10-50 concurrent requests handle kare (model size + token length pe depend). Toh roughly 20-100 GPU instances chahiye peak pe — autoscaled.

**High-level design:** User → API Gateway (auth, rate limit) → Load Balancer → fleet of LLM inference servers (vLLM on GPU, K8s pods) → response streamed back. Side: prompt/response cache (Redis), monitoring, model loaded from S3/registry.

**Deep dive:**
- **Inference layer:** vLLM (PagedAttention + continuous batching for throughput) on GPU nodes, K8s pods with GPU scheduling. Model from S3/registry loaded at pod startup.
- **Caching:** semantic/exact prompt cache (Redis) — repeated queries pe LLM call bachao (cost + latency). Big lever.
- **API Gateway:** auth (API keys/OIDC), rate limiting (per-user quotas), request validation.
- **Autoscaling:** K8s HPA on GPU utilization / queue depth. Scale-out on load, scale-in idle.
- **Streaming:** Server-Sent Events (SSE) — token-by-token to user (perceived latency down).

**Bottlenecks + scale:** GPU is bottleneck (expensive, limited). Scale via more GPU pods + autoscale. Batching (vLLM) maximizes per-GPU throughput. Cache reduces load. Queue (SQS) for overflow/spike absorption.

**Trade-offs:** Self-host vLLM (cheaper at scale, full control, ops burden) vs Bedrock (managed, less ops, cost/control trade-off) — for 1000 QPS steady, self-host likely cheaper beyond breakeven; for spiky, managed/serverless better. Batching improves throughput but adds latency — tune batch size.

**Failure:** GPU node fail → K8s reschedules, LB health-check routes away. Model load fail → pod not-ready, no traffic. Overload → queue + rate limit + graceful degradation (cached/smaller model fallback). Multi-AZ for zone failure.

**Interview line:** "API Gateway (auth/rate-limit) → LB → vLLM GPU fleet on K8s (autoscaled, batched) → streamed response, with Redis prompt caching. GPU is bottleneck — scale via autoscale + batching + cache. Self-host vs Bedrock trade-off on scale/cost. Multi-AZ + health-check failover."

---

## CASE STUDY 2: Design a RAG System at Scale

**Question:** "Design a RAG system for enterprise document Q&A — millions of documents, thousands of users."

**Requirements:** Functional — user question → relevant docs se grounded answer with citations. Non-functional — millions of docs (ingestion scale), thousands of concurrent users (query scale), fresh docs (ingestion pipeline), low latency, accuracy (good retrieval).

**Scale:** Millions of docs → chunked → maybe 10s of millions of embeddings in vector DB (needs sharding). Query QPS moderate but each = embed + retrieve + LLM.

**High-level design:** Two pipelines. **Ingestion (offline, async):** documents → queue (SQS) → workers (chunk → embed → store in vector DB). **Query (online):** user query → embed → vector DB retrieve top-k → rerank → assemble context + prompt → LLM → answer with citations. Plus caching, monitoring, eval.

**Deep dive:**
- **Ingestion:** event-driven (new doc → SQS → worker pool, autoscaled). Chunking strategy (size + overlap tuned for retrieval quality). Embedding (Bedrock Titan or self-hosted BGE — latency/cost trade-off). Batch embed for throughput.
- **Vector DB:** OpenSearch/Pinecone — sharded (millions of vectors), replicated (HA + read scale). Index type (HNSW for speed).
- **Retrieval:** top-k similarity search, then rerank (cross-encoder for relevance) — quality up.
- **LLM:** vLLM/Bedrock, context assembled with retrieved chunks + citations.
- **Caching:** query embedding cache, retrieval cache, response cache.

**Bottlenecks + scale:** Vector DB (millions of vectors) — shard + replicate. Ingestion (millions of docs) — async workers autoscaled. LLM — autoscale. Embedding — batch + cache.

**Trade-offs:** Vector DB managed (Pinecone — easy, cost) vs self-host (OpenSearch — control, ops). Chunk size — small (precise but more chunks) vs large (context but less precise). Rerank adds latency but improves quality. Embedding model — bigger (accurate, slow/costly) vs smaller (fast/cheap).

**Failure:** Ingestion worker fail → SQS retry (message safe). Vector DB node fail → replica. LLM fail → fallback/cached. Bad retrieval → eval monitoring (retrieval metrics — recall, MRR from tera RAG repo) alerts.

**Interview line:** "Two pipelines — async ingestion (SQS → workers: chunk→embed→sharded vector DB) and online query (embed→retrieve→rerank→LLM with citations). Vector DB sharded+replicated, ingestion autoscaled, aggressive caching. Trade-offs: managed vs self-host vector DB, chunk size, rerank latency-vs-quality. Retrieval metrics monitored."

---

## CASE STUDY 3: Design an End-to-End MLOps Platform

**Question:** "Design an MLOps platform for a company with multiple ML teams."

**Requirements:** Multiple teams, many models, full lifecycle (train → deploy → monitor → retrain), reproducibility, governance, self-service.

**High-level design:** Central platform: **Experiment tracking + registry** (MLflow — backend RDS PostgreSQL, artifacts S3), **Feature store** (offline + online, consistency), **Pipeline orchestration** (SageMaker Pipelines/Airflow/Kubeflow), **Training infra** (GPU on-demand/spot, K8s/SageMaker), **Serving infra** (endpoints, autoscaled), **CI/CD** (GitHub Actions → train → eval gate → register → deploy), **Monitoring** (drift, performance, infra — Prometheus/Grafana/CloudWatch).

**Deep dive:**
- **Multi-tenancy:** teams isolated (namespaces/accounts), shared platform, RBAC, cost attribution (tags).
- **Reproducibility:** MLflow (params/metrics/model+env), MLproject/containers, data versioning (DVC).
- **Governance:** model registry (versions, approvals, champion/challenger), audit trail, lineage.
- **Self-service:** teams apna experiment/train/deploy kar sakein via standard pipelines (platform team infra sambhaale).

**Trade-offs:** Managed (SageMaker end-to-end — fast, AWS-locked) vs open-source (MLflow+Kubeflow on K8s — flexible, ops burden). Central platform (consistency, governance) vs team autonomy (flexibility). Build vs buy.

**Failure/Ops:** RDS Multi-AZ, S3 durable, pipeline retries, monitoring + alerting, DR (backups, cross-region).

**Interview line:** "Central self-service platform — MLflow (RDS+S3) for tracking/registry, feature store (consistency), orchestration (SageMaker Pipelines/Kubeflow), GPU training (spot), autoscaled serving, CI/CD with eval gate, monitoring (drift+infra). Multi-tenant with RBAC + cost attribution + governance. Managed vs open-source trade-off."

---

## CASE STUDY 4: Design Real-Time Fraud Detection (Classic ML)

**Question:** "Design a real-time fraud detection system — millions of transactions, <100ms latency."

**Requirements:** Every transaction scored real-time, <100ms (tight!), high throughput, high availability (24/7 money), low false-positives.

**High-level design:** Transaction → API → feature lookup (online feature store, low-latency) → model inference (fast model, not LLM) → score → decision (approve/flag/block). Async: log for retraining, drift monitoring.

**Deep dive:**
- **Latency <100ms:** online feature store (Redis/DynamoDB — sub-ms lookup), lightweight model (not deep — gradient boosting/small NN), model in-memory on serving instances, caching.
- **Features:** online store (real-time — recent transaction velocity) + offline (historical — from feature store, skew avoided).
- **Throughput:** horizontal scale, LB, stateless inference servers.
- **Retraining:** fraud patterns evolve (concept drift) — frequent retraining, monitoring critical.

**Trade-offs:** Model complexity vs latency (bigger = accurate but slow — <100ms forces lightweight). False-positive vs false-negative balance (block real fraud vs annoy legit users — threshold tuning). Real-time vs batch features.

**Failure:** Feature store down → fallback (cached/default features + flag for review). Model down → conservative default (flag suspicious). Never block all (business impact).

**Interview line:** "Transaction → online feature store (sub-ms) → lightweight model (latency <100ms) → decision. Online+offline features (skew-free), horizontal scale + LB, in-memory model. Concept drift → frequent retrain + monitoring. Trade-off: model complexity vs latency, FP vs FN threshold. Graceful fallback on failure."

---

## CASE STUDY 5: Design a Multi-Model LLM Router / Gateway

**Question:** "Design a gateway that routes requests to different LLMs (cost/quality optimization)."

**High-level:** Request → gateway → routing logic (query complexity/type → model selection: simple→cheap/small, complex→powerful) → selected LLM (Bedrock/self-hosted) → response. Plus caching, cost tracking, fallback.

**Deep dive:** Routing — rule-based (keywords/length) or ML-based (classifier). Cost tracking per model/request (tags). Caching (semantic). Fallback (primary model down → secondary). Rate limiting per model/user. Observability (which model, cost, latency per request).

**Trade-offs:** Routing complexity vs benefit. Cheap model risk (quality) vs cost saving. Managed vs self-host per model.

**Interview line:** "Gateway with routing logic (complexity → model tier: cheap for simple, powerful for complex), semantic caching, per-model cost tracking, fallback, rate limiting, observability. Cost/quality optimization via tiering. Trade-off: routing complexity vs savings."

---

## CASE STUDY 6: Design Model Training Infrastructure (Large-Scale)

**Question:** "Design infra to train large models (distributed, GPU clusters)."

**High-level:** Data (S3) → distributed training job (multi-GPU/multi-node) → checkpoints (S3) → trained model → registry. Orchestrated (SageMaker Training/K8s jobs).

**Deep dive:** Distributed training (data parallelism — DDP, ya model parallelism — FSDP/tensor for huge models). Inter-GPU (NVLink intra-node, InfiniBand inter-node — bottleneck). Checkpointing (resume on failure/spot interruption). Spot instances (70-90% cheaper, interruption-tolerant via checkpoints). Data loading (efficient — S3 streaming, avoid GPU starvation). Orchestration + monitoring (GPU utilization, training metrics — MLflow).

**Trade-offs:** Spot (cheap, interruptions) vs on-demand (reliable, costly). Data vs model parallelism (model size dictates). GPU type (A100 fast/costly vs cheaper).

**Interview line:** "Distributed training — data parallelism (DDP) or model/tensor parallelism (FSDP for huge), high-speed interconnect (NVLink/InfiniBand — bottleneck), checkpointing (resume + spot-safe), spot instances (cost), efficient S3 data loading (avoid GPU starvation), MLflow tracking. Trade-off: spot vs on-demand, parallelism strategy by model size."

---

## Final: Interview Approach Cheatsheet

Jab koi bhi design question aaye:
1. **Ruko, clarify karo** — "Let me understand requirements" (functional + scale numbers). Jaldi solution mat de.
2. **Scale estimate** — rough QPS/storage/GPU count.
3. **High-level pehle** — simple boxes+arrows, phir detail.
4. **Bottleneck identify** — usually GPU (LLM) ya DB. Scale that.
5. **Trade-offs bolo** — har choice pe "X kyunki Y, lekin Z trade-off" — yeh seniority.
6. **Failure discuss** — "agar yeh fail ho to..." — production maturity.
7. **Tera edge use kar** — jahan networking/infra/security aaye, depth dikha (LB, VPC, HA, cost) — yahan tu shine karega.

**Golden interview line (opening):** "Main pehle requirements aur scale clarify karunga, phir high-level design, phir components deep-dive with trade-offs, aur bottleneck/failure handling. Sound good?"
