# SD-04: Production Concerns (Zero to Expert)

> System banaya to theek, par production me chalana alag khel. Yeh woh cheezein jo architect ko "production-ready" banati — aur tera infra/ops background yahan sabse zyada chamakta.

---

## 1. Monitoring & Observability (ML-specific)

Normal software monitoring (CPU, memory, latency, errors) to tu jaanta. ML me **extra layers** hote. Teen levels samajh: **Infrastructure monitoring** (GPU utilization, memory, latency, throughput — Prometheus/Grafana/CloudWatch), **Model performance monitoring** (accuracy, prediction distribution — kya model sahi predict kar raha production me), aur **Data monitoring** (input data quality, distribution). ML me ek unique problem — model **silently degrade** ho sakta bina koi error diye (predictions galat hone lagti par system "up" dikhta). Isiliye ML monitoring me prediction quality track karna padta, sirf uptime nahi. Observability ke teen pillars: metrics (numbers), logs (events), traces (request ka poora journey — especially agents/RAG me har step). OpenTelemetry standard, Jaeger tracing.

**Networking analogy:** Infra monitoring = SNMP/interface counters (tu jaanta). Model monitoring = ek extra layer — jaise link "up" hai par packet loss/corruption ho raha (silently degrading). Traces = end-to-end path visibility (traceroute jaisa).

**Interview line:** "ML monitoring 3 layers — infra (GPU/latency, Prometheus), model performance (accuracy/prediction distribution — silent degradation catch), data quality. Observability: metrics+logs+traces (OpenTelemetry/Jaeger). Uptime kaafi nahi, prediction quality track karo."

---

## 2. Model Drift (ML-unique, IMPORTANT)

Yeh ML ka sabse unique production problem hai aur interview me aata. Model train hota historical data pe, par duniya badalti rehti — isliye time ke saath model **drift** hota (accuracy girti). Do types: **Data drift** (input data distribution badal gaya — e.g., naye tarah ke users aane lage jo training me nahi the) aur **Concept drift** (input-output relationship khud badal gaya — e.g., fraud patterns evolve). Detect kaise: production predictions aur actuals compare (jab actuals milte), input distribution statistically compare (training vs live), tools jaise Evidently. Handle: alert on drift → trigger **retraining** (automated pipeline se naya model, evaluate, deploy). Yeh MLOps loop ka critical part.

**Networking analogy:** Drift = network baseline se deviation — traffic patterns badle, purana capacity plan invalid ho gaya. Monitor deviation, re-plan (retrain) jab threshold cross.

**Interview line:** "Model drift — data drift (input distribution shift) ya concept drift (input-output relationship change). Time ke saath accuracy girti. Detect via prediction-vs-actual + distribution comparison (Evidently). Handle: alert → automated retrain → evaluate → redeploy."

---

## 3. Security (Tera Strength — yahan chamak)

Yeh tera CSE background ka maidan. ML/LLM system me security layers: **Network** (VPC, private subnets, security groups — model servers internet-facing nahi, ALB/API Gateway peeche), **Authentication/Authorization** (API auth, IAM roles least-privilege, no long-lived keys), **Data security** (encryption at-rest KMS + in-transit TLS, PII handling, data access controls), **Model security** (model artifacts protected, endpoint auth), aur **LLM-specific threats** — prompt injection (user malicious prompt se model manipulate), data leakage (model training data ya context leak kare), jailbreaks. Iske liye **guardrails** (input/output filtering — Bedrock Guardrails), rate limiting, input validation. Secrets management (Secrets Manager, tera MCP project me JWT/Secrets tha).

**Networking analogy:** Yeh literally tera domain — defense in depth, VPC segmentation, least privilege, encryption, WAF-style filtering (guardrails = WAF for LLM). Tu isko interview me sabse strong bol sakta.

**Interview line:** "Security defense-in-depth — VPC/private subnets, IAM least-privilege + no long-lived keys, KMS encryption at-rest + TLS in-transit, PII handling. LLM-specific: prompt injection, data leakage, jailbreaks — guardrails (input/output filter), rate limiting, validation. Secrets Manager."

---

## 4. High Availability & Disaster Recovery

Production system down nahi ho sakta. HA (SD-01) — multi-AZ redundancy, LB, health checks, auto-scaling, failover. **DR** (Disaster Recovery) ek step aage — poora region ya major failure me recovery. Concepts: **RTO** (Recovery Time Objective — kitni jaldi recover), **RPO** (Recovery Point Objective — kitna data loss acceptable). Strategies: multi-region deployment (active-active ya active-passive), backups (RDS automated, S3 versioning + cross-region replication), model artifacts replicated. ML-specific: model registry backed up, feature store replicated, retraining capability preserved. Cost vs resilience trade-off — multi-region mehenga, criticality ke hisaab se decide.

**Networking analogy:** Yeh bilkul tera — multi-site failover, RTO/RPO tu jaanta (DR planning), active-active vs active-passive, backup/replication. ML components ke saath same principles.

**Interview line:** "HA = multi-AZ redundancy + LB + failover. DR = region-level — RTO/RPO define, multi-region (active-active/passive), backups (RDS/S3 cross-region), model registry + feature store replicated. Cost vs resilience per criticality."

---

## 5. Cost Optimization (FinOps — Architect responsibility)

AI infra mehenga (GPU!), cost optimization architect ki key responsibility, aur interview me valued. Levers: **Compute** — spot instances for training (70-90% saving, checkpoint for interruption), right-sizing (over-provisioned mat), auto-scaling (idle pe scale-down), serverless for spiky. **GPU** — maximize utilization (batching), quantization (chhote instances), scale-to-zero jab idle. **Storage** — S3 lifecycle (old → Glacier), cleanup unused artifacts. **LLM-specific** — model tiering, caching (repeated calls bachao), self-host vs managed breakeven analysis. **Monitoring** — cost dashboards (CloudWatch/Cost Explorer), per-model/per-team attribution (tags), budgets + alerts. Yeh tera existing cost/FinOps background se strong connect.

**Interview line:** "Cost — spot for training (+checkpoint), right-size + autoscale, GPU utilization (batching), S3 lifecycle, LLM caching + model tiering + self-host breakeven. Monitor via Cost Explorer + tag-based attribution + budgets/alerts. Continuous FinOps."

---

## 6. MLOps CI/CD Pipeline (End-to-End Automation)

Sab manual nahi ho sakta scale pe — automation chahiye. ML CI/CD (aksar CT bhi — Continuous Training): code commit → **CI** (test, lint, build) → **training** (automated, on new data/code) → **evaluation** (metrics, baseline gate — pass tabhi aage) → **model registry** (register, champion/challenger) → **CD** (deploy — canary/blue-green) → **monitoring** (drift) → drift pe **retraining trigger** (loop). Tools: GitHub Actions/GitLab CI (tera MCP project me GitLab CI tha), SageMaker Pipelines, Kubeflow, Argo. IaC (Terraform/CDK — tera strength) infra provision. Yeh MLflow (tracking/registry) + serving + orchestration ko tie karta.

**Networking analogy:** Yeh IaC/automation jaisa jo tu karta — validate → apply → verify → monitor → auto-remediate. ML me "apply" = train+deploy, "remediate" = retrain.

**Interview line:** "ML CI/CD/CT — commit → test → train → evaluate (baseline gate) → register (champion/challenger) → deploy (canary/blue-green) → monitor (drift) → retrain loop. GitHub Actions/SageMaker Pipelines, IaC for infra. Ties MLflow + serving + orchestration."

---

## 7. Scalability & Bottleneck Analysis (Design thinking)

Har design me interviewer poochta "scale kaise karega, bottleneck kahan." Systematic soch: **Identify bottleneck** (kaunsa component pehle saturate — usually DB, ya GPU in ML). **Scale that** — stateless components horizontal (add servers + LB), stateful (DB — replicas for read, shard for write), GPU (more instances + autoscale), storage (S3 scales itself). **Remove bottleneck via** — caching (load reduce), async/queue (spike absorb), CDN (edge), read replicas. **Capacity planning** — QPS × latency se instances calculate. Har layer independently scalable rakhna (decoupled architecture).

**Interview line:** "Bottleneck identify (DB/GPU usually), stateless horizontal + LB, stateful replicate/shard, GPU autoscale, cache to reduce load, queue to absorb spikes. Decoupled layers independently scalable. Capacity plan from QPS × latency."

---

## Interview Q&A (Production)

**Q: ML monitoring normal se kaise alag?** — "Extra layers — model performance (accuracy/prediction distribution, silent degradation) + data quality, sirf infra/uptime nahi. Metrics+logs+traces."

**Q: Model drift kya, kaise handle?** — "Data drift (input shift) ya concept drift (relationship change) — accuracy girti. Detect via prediction-vs-actual + distribution comparison. Handle: alert → automated retrain → redeploy."

**Q: LLM security threats?** — "Prompt injection, data leakage, jailbreaks. Plus standard — VPC, IAM least-priv, encryption. Guardrails (input/output filter), rate limit, validation."

**Q: RTO vs RPO?** — "RTO = recovery time target, RPO = acceptable data loss. Multi-region + backups + replication per criticality."

**Q: AI infra cost kaise optimize?** — "Spot training, right-size+autoscale, GPU utilization (batching), quantization, S3 lifecycle, LLM caching + tiering, self-host breakeven. Monitor + tag attribution + budgets."

**Q: System scale kaise, bottleneck kahan?** — "Identify saturating component (DB/GPU), stateless horizontal, stateful replicate/shard, cache + queue, autoscale. Decoupled = independently scalable."
