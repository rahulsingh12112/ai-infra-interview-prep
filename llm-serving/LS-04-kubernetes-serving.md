# LS-04: Kubernetes Serving — GPU, Autoscaling, Production (Zero to Expert)

> Tera K8s base (MCP project) + GPU serving. Self-hosted LLM serving ka production home. Yeh tera strength area — leverage kar.

---

## 1. Kyun Kubernetes for ML Serving

K8s ML serving ka de-facto platform hai kyunki yeh **container orchestration** deta jo serving ko chahiye — auto-scaling, self-healing (pod mare to restart), rolling updates, resource management, multi-tenancy. Self-hosted LLM serving (vLLM) production me aksar K8s pe chalta — GPU nodes, vLLM pods, autoscaling, LB. Tera MCP project (Helm/ArgoCD/GitLab CI) already K8s hai — yeh knowledge directly transfer hota.

**Networking analogy:** K8s = ek self-managing network fabric — nodes (devices) add/remove, traffic (pods) auto-distribute, failure pe auto-reroute (reschedule), config declarative (jaise IaC). Tu yeh orchestration mindset jaanta.

**Interview line:** "K8s for ML serving — container orchestration: autoscaling, self-healing, rolling updates, resource mgmt, multi-tenancy. Self-hosted LLM (vLLM) production home. GPU nodes + pods + autoscale + LB."

---

## 2. GPU Scheduling on Kubernetes

K8s by default GPU nahi jaanta — **NVIDIA device plugin** (DaemonSet) install karna padta jo GPUs ko K8s resources (`nvidia.com/gpu`) ke roop me expose karta. Phir pod spec me GPU request:
```yaml
resources:
  limits:
    nvidia.com/gpu: 1   # 1 GPU is pod ko
```
**GPU node pools** — GPU instances (g5/p4d) ka alag node group, aur **taints/tolerations** se ensure ki sirf GPU-needing pods wahan schedule hon (non-GPU workload GPU node pe waste na kare). **Node selectors/affinity** se right GPU type pe schedule. Ek GPU aksar ek pod ko dedicated (LLM), ya MIG (Multi-Instance GPU) se ek GPU ko split (chhote models).

**Networking analogy:** GPU scheduling = specialized resource allocation — GPU nodes = premium links, taints = access control (sirf authorized traffic), affinity = routing to right resource. Capacity planning jaisa.

**Interview line:** "K8s GPU — NVIDIA device plugin exposes nvidia.com/gpu, pod requests it, GPU node pools (g5/p4d) with taints/tolerations (GPU workload only), node affinity for right type. 1 GPU/pod (LLM) or MIG split (small models)."

---

## 3. Autoscaling (3 levels)

K8s serving me teen autoscaling layers: **HPA (Horizontal Pod Autoscaler)** — metric (CPU/GPU utilization, ya custom jaise queue depth/requests-per-pod) pe pods scale up/down. **VPA (Vertical Pod Autoscaler)** — pod ke resource requests adjust. **Cluster Autoscaler** — jab pods schedule nahi ho paate (no capacity), naye nodes add (aur idle nodes remove). LLM serving me HPA on GPU utilization ya request queue depth, + Cluster Autoscaler for GPU nodes. **KEDA** (event-driven autoscaling) — queue length jaise external metrics pe scale (advanced). Challenge: GPU nodes slow to provision (minutes) — pre-warming ya buffer capacity.

**Interview line:** "Autoscaling 3 levels — HPA (pods on GPU-util/queue-depth), VPA (resource tuning), Cluster Autoscaler (nodes). KEDA for event-driven. LLM: HPA on GPU-util/queue + cluster autoscaler for GPU nodes. GPU node provisioning slow — buffer/pre-warm."

---

## 4. KServe (K8s-native Model Serving)

KServe ek K8s-native serving framework hai jo model serving ko simplify karta — declarative `InferenceService` CRD me model do, KServe deployment/autoscaling/routing sambhaale. Features: **scale-to-zero** (idle pe pods 0 — cost saving, serverless-like on K8s), **canary rollout** (traffic split), **multi-framework** (sklearn/PyTorch/vLLM), **request batching**. LLM ke liye KServe + vLLM backend. Yeh SageMaker jaisa managed-feel deta par apne K8s pe (portable, no vendor lock-in).

**Interview line:** "KServe — K8s-native serving, InferenceService CRD (declarative), scale-to-zero (cost), canary rollout, multi-framework + vLLM. SageMaker-like on own K8s — portable, no lock-in."

---

## 5. Production Serving Architecture on K8s

Ek complete picture: **Ingress/API Gateway** (auth, rate-limit, TLS) → **Service/LB** → **vLLM pods** (GPU nodes, HPA autoscaled) → model from **S3/registry** (init container loads). Side: **Redis** (prompt cache), **Prometheus + Grafana** (GPU/latency/throughput monitoring), **queue** (SQS/Kafka for spike/batch). **Helm** charts for deployment (tera MCP project), **ArgoCD** GitOps (declarative, auto-sync — tera stack), **GitLab CI** for pipeline. Multi-AZ node pools for HA. Namespaces + RBAC for multi-tenancy.

**Networking analogy:** Yeh ek complete network architecture jaisa — edge (ingress/gateway), distribution (LB/service), access (pods on nodes), management (monitoring), automation (GitOps/Helm). Tu yeh layered design jaanta.

**Interview line:** "Production K8s serving — Ingress (auth/rate-limit/TLS) → Service/LB → vLLM pods (GPU nodes, HPA) → model from S3. Redis cache, Prometheus/Grafana monitoring, queue for spikes. Helm + ArgoCD (GitOps) + GitLab CI. Multi-AZ HA, namespaces+RBAC multi-tenancy."

---

## 6. EKS-Specific (AWS Managed K8s — tera stack)

AWS pe K8s = **EKS** (managed control plane). GPU serving ke liye: EKS + GPU node groups (managed node groups with g5/p4d), NVIDIA plugin, Cluster Autoscaler (ya Karpenter — AWS ka smarter autoscaler, faster node provisioning), ALB Ingress Controller (AWS LB), IRSA (IAM Roles for Service Accounts — pods ko S3/ECR access securely). Model artifacts S3 se, images ECR se. Yeh tera existing AWS + K8s knowledge ka perfect intersection.

**Interview line:** "EKS (managed K8s) — GPU managed node groups (g5/p4d), NVIDIA plugin, Karpenter (fast autoscale), ALB Ingress, IRSA (secure pod IAM for S3/ECR). Model S3, images ECR. AWS + K8s intersection."

---

## Interview Q&A (K8s Serving)

**Q: K8s pe GPU kaise?** — "NVIDIA device plugin exposes nvidia.com/gpu, pod requests it, GPU node pools + taints/tolerations, node affinity. 1 GPU/pod or MIG split."

**Q: Autoscaling LLM serving?** — "HPA (GPU-util/queue-depth), Cluster Autoscaler (GPU nodes), KEDA (event-driven). GPU nodes slow to provision — buffer/Karpenter."

**Q: KServe kya?** — "K8s-native serving, InferenceService CRD, scale-to-zero, canary, multi-framework+vLLM. Managed-feel on own K8s."

**Q: EKS GPU serving stack?** — "EKS + GPU node groups + NVIDIA plugin + Karpenter + ALB Ingress + IRSA. Model S3, images ECR."

**Q: Managed (SageMaker) vs self-host (EKS+vLLM)?** — "SageMaker less ops/fast, EKS+vLLM cheaper-at-scale/full-control/portable but ops. Team + scale + control decides."
