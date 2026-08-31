# AI Infrastructure — Interview Prep Notes

> Study notes for AI Infrastructure Architect / Senior ML Platform Engineer roles.
> Hinglish, structured, interview-focused. Zero-to-expert coverage of core AI infra topics.
>
> _Note: These are personal study notes compiled with AI assistance from public sources and documentation. Verify code/commands against official docs before production use._

## Contents

### 📐 System Design (`system-design/`)
System design foundations → ML system design → LLM/GenAI design → production concerns → 6 solved interview case studies.
- SD-01 Foundations (scalability, latency, HA, CAP, LB, caching, queues, DB scaling)
- SD-02 ML System Design (training vs inference, serving patterns, feature store, deployment)
- SD-03 LLM & GenAI (LLM serving, vLLM, RAG at scale, agents, cost/latency, Bedrock)
- SD-04 Production (monitoring, drift, security, HA/DR, cost, MLOps CI/CD)
- SD-05 Case Studies (LLM serving platform, RAG, MLOps platform, fraud detection, LLM router, training infra)

### 🚀 LLM Serving (`llm-serving/`)
Model serving fundamentals → LLM serving deep (vLLM) → AWS serving → Kubernetes serving → optimization.
- LS-01 Serving Fundamentals (patterns, metrics, formats, servers)
- LS-02 LLM Serving Deep (vLLM, PagedAttention, continuous batching, quantization, parallelism)
- LS-03 AWS Serving (SageMaker 4 options, Bedrock, managed vs self-host)
- LS-04 Kubernetes Serving (GPU scheduling, autoscaling, KServe, EKS)
- LS-05 Optimization & Interview (cost/latency, benchmarking, full design walkthrough)

### 🎮 GPU Infrastructure (`gpu/`)
GPU fundamentals → training/inference → cloud GPU & cost → interview scenarios.
- GP-01 Fundamentals (CPU vs GPU, VRAM math, CUDA, types, precision)
- GP-02 Training & Inference (utilization, multi-GPU parallelism, interconnect)
- GP-03 Cloud GPU & Cost (AWS instances, spot, MIG, optimization)
- GP-04 Interview Scenarios (Q&A + walkthroughs)

## How to Use
Each file: interview one-liner → layer-by-layer concept → code/config → analogy → interview Q&A → gotchas. Read in order within each topic.
