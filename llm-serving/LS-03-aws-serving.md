# LS-03: AWS Serving — SageMaker & Bedrock (Zero to Expert)

> Tera AWS stack. Yeh interview me directly poochha jaayega AWS role me. SageMaker endpoints, Bedrock, deployment options.

---

## 1. SageMaker Inference — 4 Options (yeh RATTA)

SageMaker model serving ke chaar options deta, aur architect ko pata hona kaunsa kab:

**Real-Time Inference** — persistent endpoint, always-on, low-latency, auto-scaling. Steady traffic + low-latency chahiye to yeh (chatbot, real-time API). Cost: always-on (idle bhi charge).

**Serverless Inference** — traffic pe scale (scale-to-zero jab idle), no infra manage. Spiky/intermittent traffic ke liye ideal — idle pe cost nahi. Trade-off: cold-start latency (pehli request slow after idle).

**Asynchronous Inference** — bade payloads, lambi processing (queue-based). Request queue me, process, result S3 me. Near-real-time, bade inputs (jaise long documents).

**Batch Transform** — bulk offline predictions (poora dataset ek saath). No endpoint, job-based. Latency irrelevant, throughput.

**Decision:** steady+low-latency → Real-Time; spiky → Serverless; bade payload/long → Async; bulk offline → Batch.

**Interview line:** "SageMaker 4 inference options — Real-Time (persistent, low-latency, steady), Serverless (scale-to-zero, spiky, cold-start), Async (queue, big payloads/long), Batch Transform (bulk offline). Choose by traffic + latency + payload."

---

## 2. SageMaker Real-Time Endpoint (Deep)

Real-time endpoint ke concepts: endpoint ke peeche ek ya zyada **instances** (compute — CPU/GPU), **endpoint config** (instance type, count, model), **auto-scaling** (target metric jaise invocations-per-instance pe scale), **multi-model endpoints** (ek endpoint pe kai models — cost saving), **multi-variant** (A/B testing — do model versions, traffic split). Deployment safe ke liye — **blue-green / canary** (SageMaker deployment guardrails). Model S3 se load hota, container (built-in ya custom/BYOC).

**Interview line:** "Real-time endpoint — instances (CPU/GPU) behind it, endpoint config, auto-scaling on invocations-per-instance, multi-model (cost) / multi-variant (A/B). Deployment guardrails (canary/blue-green). Model from S3, built-in or BYOC container."

---

## 3. LLM on SageMaker (LMI / DJL)

Bade LLMs SageMaker pe serve karne ke liye **LMI (Large Model Inference) containers** — yeh vLLM/TensorRT-LLM/DeepSpeed backend use karte, tensor parallelism, quantization support. Tu HuggingFace model deploy kar sakta with LMI container, ya JumpStart se pre-configured models (Llama, etc. one-click). SageMaker GPU instances (ml.g5, ml.p4d) pe. Yeh managed-ish LLM serving — infra SageMaker sambhaale, tu config de.

**Interview line:** "LLM on SageMaker — LMI containers (vLLM/TensorRT backend, tensor parallelism, quantization), JumpStart (pre-configured Llama etc. one-click), GPU instances (g5/p4d). Managed LLM serving."

---

## 4. Bedrock (Fully Managed — No Infra)

Bedrock = fully managed foundation models, **zero infra**. Tu API call karta, AWS sab sambhaale. Models: Claude (Anthropic), Titan (Amazon), Llama, etc. Two modes: **On-demand** (pay-per-token, no commitment — variable/low traffic) aur **Provisioned Throughput** (guaranteed capacity, committed — high steady traffic, cost-predictable). Bedrock ke aage services: **Agents** (managed agentic), **Knowledge Bases** (managed RAG), **Guardrails** (safety/PII/toxicity filtering). Yeh AWS-native GenAI ka easiest path.

**Interview line:** "Bedrock = fully managed FMs (Claude/Titan/Llama), zero infra. On-demand (pay-per-token, variable) or Provisioned Throughput (guaranteed, steady high-traffic). + Agents, Knowledge Bases, Guardrails. Easiest AWS GenAI."

---

## 5. THE BIG DECISION — Managed vs Self-Hosted

Yeh interview me zaroor aata — Bedrock (managed) vs vLLM-on-EKS (self-host)? Trade-off samajhna architect ki maturity dikhata:

**Bedrock (managed):** Pros — zero infra/ops, fast to market, auto-scale, security/compliance built-in. Cons — cost per-token (high volume pe mehenga), less control (model choice limited to available, no custom fine-tune serving easily), vendor lock-in, data leaves your control (though Bedrock keeps in-account).

**Self-host (vLLM on EKS/EC2):** Pros — cheaper at high scale (beyond breakeven), full control (any model, custom), no per-token cost (fixed GPU cost), data stays in your infra. Cons — ops burden (GPU management, scaling, updates), slower to market, need expertise.

**Decision framework:** Low/variable volume + fast-to-market + limited ML-infra team → Bedrock. High steady volume + cost-sensitive + control needed + have infra team → self-host. Often **hybrid** — Bedrock for spiky/experimentation, self-host for high-volume steady workloads.

**Networking analogy:** Managed vs self-host = managed cloud service vs run-your-own. Jaise managed VPN service vs apna infra pe VPN — convenience/ops vs cost/control at scale.

**Interview line:** "Bedrock (managed) — zero ops, fast, per-token cost (mehenga at scale), less control. Self-host vLLM/EKS — cheaper at scale, full control, ops burden. Decision: volume + cost-sensitivity + control + team. Often hybrid — managed for spiky, self-host for high-volume steady. Breakeven analysis."

---

## 6. MLflow → SageMaker Deployment (tera MLflow doc se connect)

Tera MLflow doc (F12) me yeh cover kiya — recap: MLflow model → `mlflow sagemaker build-and-push-container` (Docker image → ECR) → `create_deployment` (SageMaker endpoint) → `invoke_endpoint` (inference). Yeh MLflow registry + SageMaker serving ko tie karta — registered champion model ko production endpoint pe.

**Interview line:** "MLflow model → build-and-push-container (ECR) → create_deployment (SageMaker endpoint) → invoke. Registry champion → production endpoint. Ties MLflow governance + SageMaker serving."

---

## Interview Q&A (AWS Serving)

**Q: SageMaker inference options?** — "Real-Time (persistent, steady low-latency), Serverless (scale-to-zero, spiky, cold-start), Async (queue, big payloads), Batch (bulk offline). By traffic+latency+payload."

**Q: LLM SageMaker pe kaise?** — "LMI containers (vLLM/TensorRT backend, parallelism, quantization) ya JumpStart (pre-configured), GPU instances (g5/p4d)."

**Q: Bedrock modes?** — "On-demand (per-token, variable) or Provisioned Throughput (guaranteed, steady). + Agents/KB/Guardrails."

**Q: Managed vs self-host?** — "Bedrock = zero ops/fast, per-token mehenga at scale, less control. Self-host = cheaper at scale/full control, ops burden. Hybrid common. Breakeven decides."

**Q: Serverless cold-start?** — "Idle ke baad pehli request slow (container/model load). Spiky traffic ke liye acceptable trade-off; steady me real-time better."
