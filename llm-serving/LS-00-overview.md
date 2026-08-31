# LLM Serving & Model Deployment — Complete Mastery (Zero to Expert)

> **Target:** AI Infrastructure Architect (₹1Cr) | **Style:** Paragraph teaching, zero to enterprise
> **Purpose:** COMPLETE LLM serving reference. Iske baad kuch aur padhne ki zaroorat nahi.

## Yeh Doc Kyun
LLM/model serving tere role ka **dil** hai — "model banaya, ab production me at scale kaise chalaoge" — yeh interview me zaroor aayega. System Design doc me theory aayi; yeh doc **implementation depth** deta — vLLM, SageMaker, K8s GPU, quantization, sab hands-on level. Tera infra background yahan directly apply hota.

## Files Roadmap
- LS-00 (yeh) — Overview
- LS-01 — Serving Fundamentals (patterns, metrics, model formats, servers)
- LS-02 — LLM Serving Deep (vLLM, batching, KV cache, quantization, parallelism)
- LS-03 — AWS Serving (SageMaker endpoints, Bedrock, deployment)
- LS-04 — Kubernetes Serving (GPU scheduling, autoscaling, KServe, production)
- LS-05 — Optimization & Interview (cost/latency, benchmarking, case Q&A)

## Serving Interview Framework
Jab "serve a model" poochhe: (1) Model type/size? (LLM vs classic — GPU?) (2) Traffic — QPS, spiky vs steady? (3) Latency budget? (4) Cost constraint? (5) Managed vs self-host? Yeh clarify karke design.
