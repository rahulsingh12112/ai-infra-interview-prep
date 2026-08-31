# GP-03: Cloud GPU & Cost Optimization (Zero to Expert)

> Tera AWS stack + FinOps. AWS GPU instances, spot, sharing (MIG), cost — architect ki key responsibility.

---

## 1. AWS GPU Instances (yeh awareness chahiye)

AWS pe GPU instances alag families me (interview me rough idea kaafi):
- **G-family (inference/graphics, cost-effective):** g4dn (T4 GPU — cheap inference), g5 (A10G — popular inference/small training), g6 (L4). Real-time inference ke liye common.
- **P-family (training, powerful):** p3 (V100 — older), p4d (A100 40GB — training workhorse), p5 (H100 — latest, big LLM training). Distributed training ke liye.
- **Inf/Trn (AWS custom chips):** Inferentia (inf2 — AWS's inference chip, cost-optimized), Trainium (trn1 — AWS's training chip). AWS-native, cheaper for supported models, par ecosystem limited (Neuron SDK).

**Decision:** inference → g5/g6 (or Inferentia for cost); training → p4d/p5; big LLM → p5 clusters. Cost vs capability balance.

**Interview line:** "AWS GPU — G-family (g5/A10G — inference, cost-effective), P-family (p4d/A100, p5/H100 — training), Inferentia/Trainium (AWS custom, cheaper for supported, Neuron SDK). Inference g5/g6, training p4d/p5."

---

## 2. GPU Cost Reality (kyun optimization critical)

GPU mehenga — rough on-demand: A10G (g5) ~$1-1.5/hr, A100 (p4d, 8-GPU) ~$32/hr, H100 (p5) more. Ek A100 cluster training days chale = hazaaron dollars. Ek always-on inference endpoint 24/7 = monthly bill bada. Isliye cost optimization architect ki **primary responsibility** — aur interview me valued (₹1Cr role me cost thinking expected). Tera FinOps/AWS cost background yahan directly value.

**Interview line:** "GPU costly — A100 8-GPU ~$32/hr, always-on inference 24/7 = big monthly. Cost optimization = architect's primary responsibility, interview-valued."

---

## 3. Cost Optimization Levers (Consolidated — RATTA)

Saare GPU cost levers ek jagah:

**Spot Instances** — interruptible, 70-90% cheaper than on-demand. Training ke liye ideal (checkpoint se resume on interruption). Inference me risky (interruption = downtime) unless fault-tolerant.

**Right-sizing** — over-provisioned GPU mat (T4 kaafi to A100 mat lo). Benchmark se decide.

**Autoscaling + scale-to-zero** — inference me traffic ke hisaab se scale, idle pe scale-down/zero (serverless).

**GPU sharing (MIG)** — ek A100 ko multiple isolated instances me split (Multi-Instance GPU — up to 7). Chhote models/light workloads ek GPU share karein — utilization up, cost/workload down.

**Utilization maximize** — batching (GPU bhar ke chale). Under-utilized GPU = waste.

**Quantization** — chhota model = sasta GPU (INT8 se A100 ki jagah A10 chal jaaye).

**Reserved/Savings Plans** — steady long-term workload pe committed pricing (on-demand se sasta).

**AWS custom chips** — Inferentia/Trainium supported models pe sasta.

**Monitoring** — cost dashboards (Cost Explorer), per-team/model tags (attribution), budgets + alerts.

**Interview line:** "GPU cost levers — spot (70-90% off, training+checkpoint), right-size, autoscale+scale-to-zero, MIG (GPU sharing, utilization up), batching, quantization (cheaper GPU), Savings Plans (steady), Inferentia/Trainium. Monitor via Cost Explorer + tags + budgets."

---

## 4. MIG (Multi-Instance GPU) — Deep

MIG NVIDIA A100/H100 feature — ek physical GPU ko **multiple isolated logical GPUs** me partition (up to 7 instances). Har instance ka apna dedicated memory + compute (isolated, not just time-sharing). Use-case: chhote models ya light inference workloads jinhe poora A100 nahi chahiye — ek A100 ko 7 me baant ke 7 workloads chalao, utilization + cost efficiency dramatically up. K8s me MIG-aware scheduling. Alternative sharing: time-slicing (less isolation) ya MPS.

**Networking analogy:** MIG = ek physical link ko multiple isolated VLANs/virtual circuits me split — har tenant ko dedicated slice, resource efficiently shared. Tu virtualization/partitioning jaanta.

**Interview line:** "MIG — ek A100/H100 ko 7 isolated logical GPUs me partition (dedicated memory+compute each). Chhote workloads share one GPU — utilization + cost up. K8s MIG-aware scheduling. Like link partitioning into isolated slices."

---

## 5. GPU on Kubernetes/EKS (LLM Serving doc se connect)

Recap (LS-04): EKS + GPU managed node groups (g5/p4d), NVIDIA device plugin (nvidia.com/gpu resource), taints/tolerations (GPU workload only), Karpenter (fast GPU node autoscaling), MIG for sharing. Cost: GPU node pools scale-in when idle, spot GPU node groups for batch/fault-tolerant, Karpenter picks cheapest fitting instance.

**Interview line:** "EKS GPU — managed node groups (g5/p4d), NVIDIA plugin, taints for GPU-only, Karpenter (fast autoscale + cheapest instance), MIG sharing, spot node groups for batch. Scale-in idle."

---

## 6. Build vs Buy — GPU Infrastructure Decision

Architect decision: apna GPU infra (EC2/EKS with GPU) vs managed (SageMaker/Bedrock)? Same managed-vs-self-host trade-off (LS-03): self-manage = cheaper at high steady scale + control, but ops burden (GPU drivers, scaling, failures). Managed = less ops, fast, but cost premium. High-volume steady LLM = self-host on EKS often cheaper; variable/experimentation = managed. On-prem GPU (very high scale, data-sensitive) rare but exists — capex vs opex.

**Interview line:** "GPU infra build-vs-buy — self-host (EKS/EC2 GPU: cheaper at scale + control, ops burden) vs managed (SageMaker/Bedrock: less ops, cost premium). High steady volume = self-host, variable = managed. On-prem for extreme scale/data-sensitive (capex)."

---

## Interview Q&A (Cloud GPU/Cost)

**Q: AWS GPU instances?** — "G-family (g5/A10G inference), P-family (p4d/A100, p5/H100 training), Inferentia/Trainium (AWS custom, cheaper). Inference g5, training p4d/p5."

**Q: GPU cost kaise optimize?** — "Spot (70-90% off, training), right-size, autoscale+scale-to-zero, MIG (sharing), batching (utilization), quantization (cheaper GPU), Savings Plans, Inferentia. Monitor+tags."

**Q: MIG kya?** — "A100/H100 ko 7 isolated logical GPUs me partition — chhote workloads share, utilization+cost efficiency up. Dedicated memory+compute each."

**Q: Spot GPU kab?** — "Training (checkpoint se resume on interruption, 70-90% cheaper). Inference risky unless fault-tolerant."

**Q: Self-host vs managed GPU?** — "Self-host (EKS GPU: cheaper at scale, control, ops burden) vs managed (SageMaker: less ops, premium). Volume + team decides."
