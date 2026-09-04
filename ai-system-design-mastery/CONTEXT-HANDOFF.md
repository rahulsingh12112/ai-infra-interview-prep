# CONTEXT HANDOFF — Rahul's AI System Design Teaching Session
> Naye chat window me ye paste karo taaki continuity bani rahe.

## Learner Profile
- **Rahul** — AWS Cloud Support Engineer II, 15 saal exp (8 networking: BGP/OSPF/MPLS, 7 AWS). **DevOps bhi jaanta** (K8s, CI/CD, IaC, containers).
- **Target:** ₹1Cr AI Infra Architect / MLOps role, interview ~2 hafte me.
- **Code deep nahi** — padh ke samajhta; infra/config/ops strong. GPU/CUDA naya tha (ab docs bane).
- macOS, conda base, AWS creds, Bedrock Claude (us-east-1). GitHub: rahulsingh12112

## TEACHING METHOD (STRICT — ye follow karo)
Rahul ne khud ye method design karwaya. Har section is format me:
1. **Paragraph pehle** — poora concept smooth kahani/flow me (context, "ho kya raha, kyun"). Koi bullet nahi. Isse mind me frame baithta.
2. **Phir Points** — wahi cheez key points me toda. Ek-ek karke, **max 3 tukde phir saans** (cognitive overload avoid).
3. **Diagram** — jahan useful, chhota flow/box (ASCII ya DI).
4. **Interview one-liner** — crisp bolne wala version.
5. **Tum bolo (active recall)** — Rahul apne shabdon me bole, phir score + feedback → aage.

### DO NOT:
- ❌ Networking/DevOps analogy **mat thopo** — Rahul khud relate kar leta (15yr exp, "baccha nahi"). Sirf agar koi analogy genuinely naya insight de to ek line.
- ❌ Ek saath 6-7 points dump mat karo (thaka deta).
- ❌ Lab steps mat do — Rahul khud lab karega alag se. **Sirf system design pe focus.**
- ❌ Over-claim mat karwao — "working vs design" clearly batao.

### DO:
- ✅ Rahul ke pace pe — wo "aage" bole tab badho.
- ✅ Galtiyan pakdo, honest score do (interview-grade).
- ✅ Hinglish, simple.
- ✅ Jab K8s padhao → AWS services (EKS/ECS/ECR/ALB/EBS/EFS/CloudFront/Karpenter) ke saath map karo — platform + K8s dono ready ho.
- ✅ Har chapter **~/AI-SYSTEM-DESIGN-MASTERY/** me save karo (baad me GitHub push).

## THE DIAGRAM (central teaching aid)
- **Design Inspector diagram: "ML design 2026"** — https://design-inspector.a2z.com/?#IML design 2026
- Ye ek AI/ML Inference Infrastructure (Realistic AWS Topology). Serving path: Users→CloudFront→Route53→ALB(least-conn)→VPC→AZ-A/B→GPU nodes(p4d, A100x4, NVLink)→EBS/EFS/S3, ElastiCache(Redis semantic cache), InfiniBand/EFA, Auto Scaling+Karpenter, Training flow.
- Neeche 4 added regions (band at y=2160): 🔐 Security (WAF/APIGW/Guardrails/Secrets+KMS), 🗄️ Data Layer (RDS/VectorDB), 📊 Observability (CloudWatch+Prom/OTel), 🔁 MLOps Loop (Drift/SageMaker Pipelines/Registry/Bedrock-alt).
- **Teaching = diagram-driven** (station by station, request flow order). Rahul ka mind topology me sochta.
- Container/Orchestration station (Docker→ECR→EKS→K8s primitives→GPU scheduling) baad me add karna hai — jab us topic pe pahunche.

## 11-STATION PLAN (flow order, har station: AI concept + DevOps/K8s angle jahan ho)
1. 👥 Users → concurrent vs total, QPS base
2. 🔐 Security band — WAF→API GW→Guardrails→Secrets/KMS
3. 🌐 CloudFront + ⚡ ElastiCache — caching, exact vs semantic
4. ⚖️ ALB — least-connections, health checks
5. 🖥️ GPU Nodes + NVLink/InfiniBand — VRAM, parallelism, comm-bound
6. 🗄️ Data Layer — RDS(metadata=SQL) + Vector DB(RAG) + S3/EFS/EBS
7. 📈 Auto Scaling + Karpenter — spike→autoscale, GPU billing, HPA/CA/Karpenter
8. 📊 Observability — 3 layers, silent degradation, traces
9. 🔁 MLOps Loop — Drift→Pipeline→Registry→Deploy (CI/CD/CT, GitOps)
10. 🔴 Training flow + 📥 SQS — offline training, async
11. 🌎 Bedrock managed alt — self-host vs managed
(+ Container/Orchestration station add karna hai in-flow)

## REFERENCE DOCS (already bane, Rahul ke)
- ~/MLFLOW-MASTERY/ (13 files), ~/SYSTEM-DESIGN-MASTERY/ (SD-00..05), ~/LLM-SERVING-MASTERY/ (LS-01..05), ~/GPU-MASTERY/ (GP-00..04)
- GitHub: github.com/rahulsingh12112/ai-infra-interview-prep (System Design, LLM Serving, GPU)
- rag-final-complete repo (RAG guides)
- SD docs padh liye is session me — content solid, Rahul ke knowledge se bane (lab/official se verify pending).

## KNOWN GAPS / MISTAKES TO DRILL
- **Data drift vs concept drift** — Rahul ne ulta kiya tha (data drift = P(X) input badla; concept drift = P(Y|X) rule badla). Drill this.
- Bulk/nightly = throughput (not latency). Spike = autoscale (not vertical). GPU billing = allocated time.
- RAG security = retrieval layer (don't retrieve what user can't see). Guardrails = content filter, NOT access control.
- Registry BEFORE deploy. Drift = monitoring output, not a pipeline stage.
- Structure discipline: "2 batao" → exactly 2 labeled, no dump.

## HONEST STATUS
- Docs = concept awareness ~70%, interview-ready ~40%.
- Plan: diagram-driven drill-down teaching (~85-90%) + beech mini-mocks + 2-3 full mocks + 1-2 hands-on labs (Rahul khud) = interview-ready.

## PROGRESS TRACKER
- [ ] Station 1: Users
- [ ] Station 2: Security band
- [ ] Station 3: CloudFront + ElastiCache
- [ ] Station 4: ALB
- [ ] Station 5: GPU Nodes + interconnect
- [ ] Station 6: Data Layer
- [ ] Station 7: Auto Scaling + Karpenter
- [ ] Station 8: Observability
- [ ] Station 9: MLOps Loop
- [ ] Station 10: Training + SQS
- [ ] Station 11: Bedrock managed
- [ ] Container/Orchestration station
- [ ] Mock interviews
- [ ] **FINAL: naya crystal-clear FULL-FLOW diagram banana** — sahi request order (User→CloudFront→WAF→API Gateway→Guardrails→ALB→GPU→data/observability/mlops), professional layout. Master diagram for interview recreate.

## NOTE on current "ML design 2026" diagram
- Security boxes conceptually grouped (WAF/APIGW/Guardrails ek band). Actual flow order alag: CloudFront(CDN) pehle → API Gateway baad. Guardrails actually LLM-call ke paas (input+output), security perimeter me nahi.
- Ye final full-flow diagram me theek karna hai.

## PROGRESS DETAIL
- Station 1 (Users): DONE 10/10. AISD-01 saved.
- Station 2 (Security): DONE 9/10 (WAF vs Guardrails nuance). AISD-02 saved.
- Station 3 (Caching): DONE 8.5/10 (CDN semantic nahi kar sakta — corrected). AISD-03 saved.
- Station 4 (ALB): DONE 10/10. AISD-04 saved.
- Station 5 (GPU): DONE. 5a 10/10, 5b 8/10 (3D not just pipeline; training=4x), 5c 9.5/10 (GB/s unit). AISD-05 saved.
- Station 6 (Data Layer): DONE 9/10 (S3 reason = designed-for-objects not just cheap). AISD-06 saved.
- Station 7 (Autoscaling): DONE 9/10 (add scheduled/predictive pre-scaling). AISD-07 saved.
- Station 8 (Observability): DONE 5.5/10 → re-drilled. WEAK SPOT: actuals delayed → prediction distribution; flag rate gira = BAD. AISD-08 saved.
- Station 9 (MLOps Loop): DONE 8.5/10 (drift = monitoring output, not stage). AISD-09 saved.
- Station 10 (Training+SQS): DONE 10/10. AISD-10 saved.
- Station 11 (Bedrock): DONE 10/10 (add trade-off articulation, avoid "obviously"). AISD-11 saved.

## ⭐ ALL 11 STATIONS COMPLETE. Next: full-flow master diagram + mock interviews.

## RE-DRILL LIST (weak spots for mock)
1. **Data drift vs concept drift** (P(X) vs P(Y|X)) — 2x confused earlier.
2. **Actuals delayed → prediction distribution first** (flag rate gira = BAD/under-detecting) — 2x missed.
3. Training VRAM = 4x (gradients + optimizer).
4. 175B fit-nahi = 3D (tensor+pipeline+data), not just pipeline.
5. Structure discipline: "N batao" → exactly N labeled.
