# SD-02: ML System Design Fundamentals (Zero to Expert)

> Ab ML-specific. Foundations (SD-01) ML components pe apply. Yeh woh cheezein jo pure software se alag hain.

---

## 1. ML System vs Normal Software System

Pehle samajh ki ML system normal software se kyun alag hai, kyunki interview me yeh base banta. Normal software me logic hardcoded hota — if-else rules programmer likhta. ML system me logic **data se seekha** jaata (model training) aur phir apply hota (inference). Iska matlab ML system me do bilkul alag phases hote hain jinke infrastructure needs alag hain: **Training** (model banana — heavy compute, GPU, batch, offline, ghante-din lagte) aur **Inference/Serving** (trained model se prediction — fast, often real-time, scale pe). Ek architect ko dono ke liye alag-alag design karna padta, aur unke beech ka bridge (model registry, deployment pipeline) bhi.

Doosri badi baat — ML system me **data aur model bhi first-class citizens** hain, sirf code nahi. Data badalta rehta (drift), model purana ho jaata (retrain chahiye), aur yeh sab track/version karna padta. Isliye ML system design me tu sirf compute/network nahi, data pipelines aur model lifecycle bhi design karta.

**Networking analogy:** Training = ek complex offline planning/simulation (network capacity planning — heavy, one-time-ish). Inference = live packet forwarding (fast, continuous, scale pe). Dono ke infra needs alag.

**Interview line:** "ML system ke do phases — training (heavy compute/GPU, batch, offline) aur inference (fast, real-time, scale). Plus data aur model lifecycle track karna. Normal software me sirf code, yahan code+data+model."

---

## 2. Training Infrastructure

Training me model data se seekhta — yeh compute-intensive hai, aksar GPU/TPU pe. Design considerations: **Single-GPU** (chhote models), **Distributed training** (bade models — data parallelism me data alag GPUs pe same model, ya model parallelism me model khud alag GPUs pe split, ya pipeline parallelism). Bade LLMs multi-node, multi-GPU pe train hote (100s of GPUs) — yahan **inter-GPU communication** (NVLink, InfiniBand) bottleneck banta, aur frameworks jaise PyTorch DDP/FSDP, DeepSpeed use hote. Training jobs ko orchestrate karne ke liye SageMaker Training, Kubernetes jobs, ya batch systems. **Checkpointing** critical — lambi training beech me fail ho to shuru se nahi, checkpoint se resume. Spot instances se cost bachao (70% saving) par interruption handle karo (checkpoint se).

**Networking analogy:** Distributed training = ek bade computation ko multiple nodes me split karke coordinate — bilkul distributed routing computation ya parallel processing jaise. Inter-GPU link = tera backbone bandwidth; bottleneck wahin banta.

**Interview line:** "Training GPU-heavy — distributed (data/model/pipeline parallelism), inter-GPU interconnect (NVLink/InfiniBand) bottleneck. Checkpointing for resume, spot instances for cost. SageMaker/K8s orchestrate."

---

## 3. Inference / Serving Patterns (CORE — tera role)

Yeh tere role ka dil hai. Inference = trained model se prediction. Char patterns samajh:

**Online (real-time) inference** — request aaya, turant prediction (latency-critical, e.g., chatbot, fraud detection). Model ek server pe loaded, REST/gRPC endpoint, LB ke peeche scaled. Latency budget tight (ms). Yeh sabse common aur challenging.

**Batch inference** — bahut saare predictions ek saath, offline (e.g., raat ko sab users ke recommendations compute). Latency matter nahi, throughput matter karta. Scheduled job, results DB me store.

**Streaming inference** — continuous data stream pe (e.g., real-time analytics, Kafka se aata data pe predict).

**Serverless inference** — spiky/unpredictable traffic ke liye (SageMaker Serverless) — idle pe cost nahi, spike pe auto-scale, par cold-start latency.

Design me decide karna: latency requirement, traffic pattern (steady vs spiky), cost, model size. Bada model + real-time + high scale = GPU servers + LB + auto-scaling + caching.

**Networking analogy:** Online inference = live traffic forwarding (low latency critical). Batch = scheduled bulk transfer (throughput). Serverless = on-demand capacity (scale-to-zero).

**Interview line:** "Serving patterns — online (real-time, latency-critical, GPU+LB+autoscale), batch (bulk offline, throughput), streaming (continuous), serverless (spiky, scale-to-zero, cold-start trade-off). Pattern traffic + latency + cost pe depend."

---

## 4. Feature Store

ML me model ko **features** chahiye (processed input signals — jaise user ka avg purchase, last login). Problem: same feature training aur inference dono me chahiye, par agar dono jagah alag tarike se compute hui to **training-serving skew** (mismatch = galat predictions). **Feature store** yeh solve karta — central place jahan features compute/store hote, dono training (batch, historical) aur serving (real-time, low-latency lookup) ke liye consistent. Do parts: offline store (historical, training ke liye, S3/warehouse) aur online store (low-latency, serving ke liye, Redis/DynamoDB). Examples: SageMaker Feature Store, Feast.

**Networking analogy:** Feature store = central config/state database (single source of truth) jise multiple systems consistently consume karte — jaise IPAM sabko same IP data deta, koi mismatch nahi.

**Interview line:** "Feature store central features — offline (training, historical) + online (serving, low-latency) — training-serving skew rokta. Consistency critical, warna galat predictions."

---

## 5. ML Pipeline Architecture (End-to-End)

Ek production ML system ek pipeline hota, sirf ek model nahi. Stages: **Data ingestion** (raw data collect) → **Data validation** (quality check) → **Feature engineering** (features banao, feature store me) → **Training** (model banao, experiments track — MLflow) → **Evaluation** (metrics, baseline gate) → **Model registry** (versioned store) → **Deployment** (serving pe) → **Monitoring** (drift, performance) → wapas retraining trigger. Yeh sab **automated + orchestrated** hona chahiye (Airflow, SageMaker Pipelines, Kubeflow). Yeh MLOps ka backbone — tera MLflow doc iska ek hissa (tracking/registry) cover karta.

**Networking analogy:** Pipeline = automated provisioning workflow — data (config) validate → apply → verify → monitor → drift pe re-provision. End-to-end automation jaisa tu IaC me karta.

**Interview line:** "Production ML = automated pipeline: ingest → validate → feature-engineer → train (track) → evaluate (gate) → register → deploy → monitor → retrain. Orchestrated via Airflow/SageMaker Pipelines. Not just a model — a lifecycle."

---

## 6. Model Deployment Strategies

Naya model deploy karna risky (galat model = bad predictions at scale). Safe strategies: **Blue-Green** (do environments, naya (green) ready karo, traffic switch, problem ho to wapas blue) — instant rollback. **Canary** (naya model ko chhote % traffic pe, monitor, dheere-dheere badhao) — risk contained. **Shadow** (naya model production traffic pe chalao par response use mat karo, sirf compare) — zero risk validation. **A/B testing** (do models, traffic split, metrics compare — business impact measure). Model registry aliases (champion/challenger — tera MLflow doc F09) yeh enable karte.

**Networking analogy:** Yeh bilkul tera domain — canary = phased config rollout (ek site pe try, phir sab), blue-green = standby device switchover, shadow = tap/mirror traffic for testing. Tu yeh jaanta.

**Interview line:** "Deployment — blue-green (instant rollback), canary (gradual, risk-contained), shadow (zero-risk validation), A/B (business metrics). Registry aliases enable. Networking ke phased rollout jaisa."

---

## 7. GPU Infrastructure Basics

ML (especially LLM) GPU pe chalta. Samajhne layak: GPU CPU se parallel compute me bahut fast (matrix ops), par mehenga aur limited. **GPU memory (VRAM)** bottleneck — bada model VRAM me fit hona chahiye (e.g., 70B model ~140GB, multiple GPUs chahiye). **GPU utilization** maximize karna cost ke liye critical (idle GPU = paisa jal raha). Techniques: batching (multiple requests ek saath GPU pe), quantization (model chhota karo — FP16/INT8, kam VRAM), model sharding (bade model multiple GPUs pe). K8s pe GPU scheduling (nvidia device plugin), node pools with GPUs.

**Networking analogy:** GPU = expensive high-bandwidth link — maximize utilization (kabhi idle na), capacity plan (VRAM = link capacity), share efficiently (batching = multiplexing).

**Interview line:** "GPU parallel-fast par mehenga + VRAM-limited. Maximize utilization (batching), reduce footprint (quantization FP16/INT8), shard big models. K8s GPU scheduling via device plugin + GPU node pools."

---

## Interview Q&A (ML System Design)

**Q: Training vs inference infra?** — "Training GPU-heavy, batch, offline, distributed + checkpointing. Inference fast, real-time/scale, LB + autoscale. Alag needs."

**Q: Online vs batch inference?** — "Online real-time latency-critical (GPU+LB). Batch bulk offline throughput. Serverless spiky (scale-to-zero, cold-start)."

**Q: Feature store kyun?** — "Central features, offline+online, training-serving skew rokta. Consistency = correct predictions."

**Q: Training-serving skew?** — "Feature training aur serving me alag compute = mismatch = galat prediction. Feature store se same source, skew avoid."

**Q: Model deploy safely?** — "Canary (gradual), blue-green (instant rollback), shadow (zero-risk validate), A/B (metrics). Registry aliases."

**Q: GPU cost kaise optimize?** — "Batching (utilization up), quantization (VRAM down), spot for training, serverless/autoscale for spiky inference, right-size."

---

## 🎓 Deep Dive & Q&A (Teacher Session — zero-se-expert, section-wise)

> Live teaching ka nichod. GPU-MASTERY se overlap wale (training infra, GPU basics, serving) short; ML-specific naye concepts deep.

### Section 1 — ML System vs Normal Software

**Normal software:** logic **hardcoded** (programmer rules likhta — "agar age>18 allow"). Code hi sab.
**ML system:** logic **data se seekha** (examples se pattern, hardcode nahi — spam detection ne khud seekha).

**2 core differences (interview opening — structured 2 points do):**
1. **Do phases (alag infra):** Training (data se seekhna — heavy GPU, batch, offline, ek-baar-ish) + Inference/Serving (predict — fast, real-time, scale, always-on). Beech me bridge (registry, deploy pipeline). Needs bilkul alag (GP-02: training=throughput/4x, inference=latency).
2. **Code + Data + Model teeno manage** (normal = sirf code): data drift hota, model purana → retrain, sab versioning. Data pipelines + model lifecycle bhi design karna.
   *(3rd valid difference: logic hardcoded vs data-learned. Koi bhi 2 labeled do.)*

**Networking analogy:** Training = offline capacity planning/simulation (heavy, one-time). Inference = live packet forwarding (fast, continuous). Planning-plane vs forwarding-plane.

**Interview one-liner:**
> "Normal: logic hardcoded. ML: logic data se seekha → 2 phases (training heavy/offline + inference fast/real-time, alag infra + bridge) aur code+data+model teeno manage (drift, retrain, versioning)."

**⚠️ STRUCTURE DRILL:** "2 differences batao" → EXACTLY 2 labeled (1)...(2)... do. Ek diya = adhoora. Number suno: "2 differences", "3 levers", "ek simple ek advanced" — utne hi, labeled.

### Section 2 — Training Infrastructure (recap, GP-02 se deep already)

- **Distributed:** data parallel (fit hai, speed — DDP), tensor (fit nahi — layer split, NVLink), pipeline (layers staged). Biggest LLM = 3D (DeepSpeed/Megatron).
- **FSDP** (Fully Sharded Data Parallel) = data-parallel + model/optimizer states GPUs me shard (memory-saving jab model ek GPU me tight).
- **Interconnect** (NVLink intra-node / InfiniBand inter-node) = bottleneck; slow → GPUs idle wait → **communication-bound**. Isliye fast link (idle avoid).
- **Checkpointing** → S3, fail/spot-interruption pe resume; spot (70-90% off) safe.
- **Orchestrate:** SageMaker Training, K8s jobs, AWS Batch.

**Interview one-liner:**
> "Training distributed: data (DDP), tensor/pipeline (big model), FSDP (sharded memory), 3D (DeepSpeed). Interconnect bottleneck → fast NVLink/InfiniBand so GPUs don't idle. Checkpoint→S3 for resume + spot saving."

**Q&A:** 100B fit nahi → tensor+pipeline (+data=3D); interconnect matters kyunki slow link → GPUs idle wait (communication-bound) → NVLink/InfiniBand chahiye.
