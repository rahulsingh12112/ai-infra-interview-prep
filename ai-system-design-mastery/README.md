# AI System Design Mastery (Zero to Hero)

> Diagram-driven, Hinglish, interview-focused study material for AI Infra / MLOps Architect role.
> Teaching format: paragraph (samajh) → points (max 3) → diagram → interview one-liner → active recall.

## 📍 Start Here
- **`00-SYLLABUS-MAP.md`** — complete 7-module syllabus, progress tracker, weak spots, recommended order.
- **`CONTEXT-HANDOFF.md`** — session context (for continuing in new chat windows).
- **`diagrams/`** — Design Inspector diagram links + XML sources.

## 📚 Chapters (Station-by-station, request-flow order)
| # | Chapter | Topic |
|---|---|---|
| 01 | `AISD-01-users.md` | Users — concurrent vs total, QPS, peak factor |
| 02 | `AISD-02-security.md` | Security band — WAF, API Gateway, Guardrails, Secrets/KMS |
| 03 | `AISD-03-caching.md` | CloudFront + ElastiCache — exact vs semantic cache |
| 04 | `AISD-04-alb.md` | ALB — least-connections, health checks |
| 05 | `AISD-05-gpu.md` | GPU — VRAM math, parallelism, interconnect |
| 06 | `AISD-06-data-layer.md` | Data — RDS/Vector DB/S3/EFS/EBS |
| 07 | `AISD-07-autoscaling.md` | Auto Scaling + Karpenter, GPU billing |
| 08 | `AISD-08-observability.md` | Observability — 3 layers, silent degradation |
| 09 | `AISD-09-mlops-loop.md` | MLOps loop — drift → pipeline → registry → deploy |
| 10 | `AISD-10-training-sqs.md` | Training (offline) + SQS (async) |
| 11 | `AISD-11-bedrock.md` | Bedrock — self-host vs managed |

## ⚠️ Weak Spots (drill in mocks)
1. Data drift P(X) vs concept drift P(Y|X)
2. Actuals delayed → prediction distribution first (flag rate gira = under-detecting = BAD)
3. Training VRAM = 4x (gradients + optimizer)
4. 175B fit-nahi = 3D parallelism (not just pipeline)
5. Structure discipline: "N batao" → exactly N labeled

## Status
11 stations done (inference platform flow). Pending: CAP, Feature Store, vLLM internals, RAG deep, DevOps/K8s module, case studies, mocks. See `00-SYLLABUS-MAP.md`.
