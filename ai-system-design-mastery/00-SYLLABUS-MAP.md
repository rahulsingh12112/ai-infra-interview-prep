# 🎯 AI/ML System Design — Zero to Hero: MASTER SYLLABUS MAP

> Complete scope. Har module + andar ke topics. Progress checkbox se track.
> Status: ✅ done | 🟡 partial | ⬜ pending

---

## 📘 MODULE 1: Core Foundations
- [🟡] 1.1 Scalability — vertical vs horizontal, autoscaling, GPU billing
- [✅] 1.2 Latency vs Throughput — trade-off, decision rule, first-token
- [✅] 1.3 Availability & HA — nines, 3 pillars, SLA/SLO/SLI
- [⬜] 1.4 CAP Theorem — C/A/P, strong vs eventual consistency
- [✅] 1.5 Load Balancing — RR/least-conn/weighted/hashing, health checks
- [✅] 1.6 Caching — hit/miss, LRU, TTL, exact vs semantic
- [✅] 1.7 Message Queues — decouple/smoothing/reliability, SQS vs Kafka
- [🟡] 1.8 Database Scaling — replication vs sharding, SQL vs NoSQL, right-store
- [✅] 1.9 Storage — Block/Object/File (EBS/S3/EFS)

## 📗 MODULE 2: ML System Fundamentals
- [🟡] 2.1 ML vs Normal Software — training vs inference, code+data+model
- [🟡] 2.2 Training Infra — data/tensor/pipeline/FSDP, interconnect, checkpoint, spot
- [⬜] 2.3 Serving Patterns — online/batch/streaming/serverless, cold-start
- [⬜] 2.4 Feature Store — training-serving skew, offline+online (IMPORTANT)
- [🟡] 2.5 ML Pipeline — stages+order, drift=output, orchestration
- [🟡] 2.6 Deployment Strategies — blue-green/canary/shadow/A-B
- [✅] 2.7 GPU Basics — VRAM math, precision

## 📙 MODULE 3: LLM & GenAI (role core)
- [🟡] 3.1 LLM Serving Challenges — VRAM, token-by-token, cost
- [⬜] 3.2 vLLM Internals — PagedAttention, continuous batching (IMPORTANT)
- [⬜] 3.3 Inference Optimization — quantization, KV cache, speculative decoding
- [⬜] 3.4 RAG at Scale — ingestion+query pipeline, chunking, rerank (IMPORTANT)
- [⬜] 3.5 Agents at Scale — loops, cost, state, tracing
- [🟡] 3.6 LLM Cost/Latency — tiering, semantic cache, compression, streaming
- [✅] 3.7 Bedrock Managed — Base/Agents/KB/Guardrails, managed vs self-host

## 📕 MODULE 4: Production Concerns (strength)
- [✅] 4.1 Monitoring — 3 layers, 3 pillars, silent degradation
- [✅] 4.2 Model Drift — data P(X) vs concept P(Y|X) [WEAK SPOT re-drill]
- [✅] 4.3 Security — defense-in-depth, LLM threats, retrieval-layer
- [⬜] 4.4 HA & DR — RTO/RPO, multi-region, registry+FS replicate
- [🟡] 4.5 Cost/FinOps — spot, right-size, tiering, tagging, breakeven
- [✅] 4.6 MLOps CI/CD/CT — automation, eval gate, retrain loop
- [⬜] 4.7 Scalability & Bottleneck — identify+scale, capacity planning

## 🛠️ MODULE 5: DevOps / Platform / AWS Architect (2nd edge)
### 5A Containers & Registry
- [⬜] 5A.1 Docker (multi-stage, CUDA base) | 5A.2 ECR | 5A.3 best practices
### 5B Kubernetes Core
- [⬜] 5B.1 Architecture | 5B.2 Primitives (Pod/Deploy/Svc/Ingress) | 5B.3 Config/Secrets
- [⬜] 5B.4 Storage (PV/PVC→EBS/EFS) | 5B.5 Networking (Ingress→ALB) | 5B.6 RBAC/limits
### 5C GPU on K8s (AI-specific)
- [⬜] 5C.1 nvidia device plugin | 5C.2 node pools/taints | 5C.3 MIG | 5C.4 requests/limits | 5C.5 vLLM on K8s
### 5D Autoscaling
- [🟡] 5D.1 HPA | 5D.2 VPA | 5D.3 Cluster Autoscaler vs Karpenter | 5D.4 KEDA | 5D.5 scale-to-zero/scheduled
### 5E EKS vs ECS vs Lambda
- [⬜] 5E.1 EKS | 5E.2 ECS/Fargate | 5E.3 Lambda | 5E.4 decision framework
### 5F CI/CD + GitOps
- [⬜] 5F.1 Pipeline | 5F.2 ArgoCD/Flux | 5F.3 Argo Rollouts/Flagger | 5F.4 CI/CD/CT
### 5G IaC
- [⬜] 5G.1 Terraform | 5G.2 CDK/CloudFormation | 5G.3 drift detection
### 5H AWS Core Services
- [⬜] 5H.1 Compute (EC2 GPU/Fargate/Lambda) | 5H.2 Networking (VPC/ALB/CF/R53/PrivateLink)
- [⬜] 5H.3 Storage (S3/EBS/EFS/FSx) | 5H.4 DB (RDS/DynamoDB/ElastiCache/OpenSearch)
- [⬜] 5H.5 Messaging (SQS/SNS/Kinesis/EventBridge) | 5H.6 Security (IAM/KMS/SM/WAF/Cognito) | 5H.7 Observability (CW/X-Ray/Container Insights)
### 5I Platform/SRE
- [⬜] 5I.1 HA/DR multi-AZ/region | 5I.2 DR patterns (pilot-light/warm-standby/active-active)
- [⬜] 5I.3 Fault tolerance (retry/circuit-breaker/degradation) | 5I.4 capacity+FinOps

## 📓 MODULE 6: Case Studies (whiteboard, framework se)
- [🟡] 6.1 LLM Serving Platform (diagram done)
- [⬜] 6.2 RAG at Scale | 6.3 MLOps Platform | 6.4 Fraud Detection
- [⬜] 6.5 Multi-model Router | 6.6 Large-scale Training Infra

## 📔 MODULE 7: Interview Skills (reflex)
- [⬜] 7.1 Framework discipline | 7.2 Estimation (numbers) | 7.3 Trade-off articulation
- [⬜] 7.4 Structure discipline | 7.5 MOCK INTERVIEWS ×3-5 (most critical)

---

## PROGRESS SNAPSHOT
```
Module 1: ~55%   Module 2: ~40%   Module 3: ~25%   Module 4: ~55%
Module 5: ~10%   Module 6: ~15%   Module 7: 0%
Overall theory: ~30-35% of full "hero" level
```

## RECOMMENDED ORDER
1. Module 1 complete (1.4 CAP, 1.8 DB depth)
2. Module 2 (2.4 Feature Store, 2.3 Serving patterns)
3. Module 3 (3.2 vLLM, 3.4 RAG, 3.3 optimization)
4. Module 5 (DevOps/K8s/AWS)
5. Module 4 baaki (4.4 DR, 4.7 bottleneck)
6. Module 6 (case studies)
7. Module 7 (mocks — throughout + end)

## WEAK SPOTS (mock me drill)
1. Data drift vs concept drift (P(X) vs P(Y|X))
2. Actuals delayed → prediction distribution (flag rate gira = under-detecting = BAD)
3. Training VRAM = 4x
4. 175B = 3D parallelism (not just pipeline)
5. Structure: "N batao" → exactly N labeled

## HONEST NOTE
Syllabus complete = theory ~100%, interview-ready ~90%. "100% guaranteed clear" myth hai — knowledge + mock + actual experience se aata.
