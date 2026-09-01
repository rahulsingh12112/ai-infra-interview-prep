# GP-04: GPU Interview Mastery — Q&A + Scenarios (Zero to Expert)

> Sab consolidate — GPU ke saare interview questions + scenario walkthroughs. Yeh padhke GPU questions confidently handle karega.

---

## 1. Quick Reference — GPU Cheatsheet (ratta)

**Memory math:** footprint ≈ params × bytes. FP32=4, FP16=2, INT8=1. 7B FP16=14GB, 70B FP16=140GB. Training ≈ 4× inference.

**GPU types:** T4/A10 (inference cheap), A100 (training+big inference, 40/80GB), H100 (latest, LLM training), L4 (inference).

**AWS:** g5 (A10G inference), p4d (A100 training), p5 (H100), Inferentia/Trainium (custom).

**Parallelism:** data (model copies), tensor/model (huge model split), pipeline (staged). LLM = tensor/3D.

**Interconnect:** PCIe (slow) < NVLink (intra-node) < InfiniBand (inter-node RDMA).

**Precision:** FP32 (full) → FP16 (standard) → INT8 (quantized, cost) → INT4 (aggressive).

**Cost levers:** spot (training), MIG (sharing), autoscale, batching, quantization, right-size, Savings Plans.

---

## 2. Common Interview Questions (complete)

**Q: CPU vs GPU AI ke liye?**
> "CPU few powerful cores (sequential), GPU thousands simple cores (massively parallel). ML = parallel matrix multiplications → GPU 10-100x faster. Isliye AI GPU pe."

**Q: VRAM constraint — 70B model?**
> "70B × 2 bytes (FP16) = 140GB. Ek 80GB GPU me nahi → 2+ GPUs (tensor parallelism) ya INT8 quantization (~70GB). Training aur zyada (4x)."

**Q: GPU utilization kaise maximize?**
> "Batching (feed GPU fully), efficient data loading (avoid starvation), continuous batching (inference/vLLM). Monitor nvidia-smi. Low util = wasted cost — biggest lever."

**Q: Multi-GPU parallelism types?**
> "Data (model copies, data split — common training), tensor/model (huge model split — LLM), pipeline (staged layers). Big LLM = 3D combo (DeepSpeed/Megatron)."

**Q: Interconnect kyun matter?**
> "Multi-GPU communication (gradient/tensor sync) bottleneck. NVLink (intra-node fast), InfiniBand (inter-node RDMA), PCIe (slow). Slow interconnect = GPUs idle waiting. NCCL optimizes."

**Q: GPU cost optimize?**
> "Spot (70-90% off, training+checkpoint), MIG (GPU sharing), autoscale+scale-to-zero, batching (utilization), quantization (cheaper GPU), right-size, Savings Plans, Inferentia. Monitor + tags + budgets."

**Q: MIG kya?**
> "A100/H100 ko 7 isolated logical GPUs me partition — dedicated memory+compute each. Chhote workloads share one GPU, utilization+cost up."

**Q: Spot instances safely?**
> "Checkpointing — state to S3, resume on interruption. Training me 70-90% cheaper. Inference risky unless fault-tolerant."

**Q: CUDA/NCCL?**
> "CUDA = NVIDIA GPU software platform, frameworks isse GPU use karte. NCCL = multi-GPU communication library (distributed training). Version compatibility critical."

**Q: Quantization trade-off?**
> "FP16→INT8: 2x less memory, faster, cheaper GPU. Minor accuracy loss (INT8 minimal). Cost inference me worth it."

---

## 3. SCENARIO WALKTHROUGHS

**Scenario 1: "Train a 13B model — GPU setup?"**
> "13B FP16 inference = 26GB, par training ~4x = ~100GB+ (weights+gradients+optimizer). Ek A100 80GB me tight → multi-GPU. Setup: 4-8× A100 (p4d) with data parallelism (DDP) if model fits per GPU, else tensor parallelism. NVLink intra-node for fast gradient sync. Checkpointing to S3 (spot-safe, 70-90% cheaper). Monitor utilization (batching to keep GPUs fed). MLflow tracking."

**Scenario 2: "Cost-optimize an inference workload spending too much on GPU."**
> "First measure — GPU utilization (nvidia-smi). Low util = under-fed → batching (continuous batching/vLLM). Then: right-size (A100 → A10 if fits, or quantize INT8 for smaller GPU), autoscale + scale-to-zero (idle pe cost nahi), MIG if small models (share one GPU), Savings Plans if steady, caching (repeated queries no GPU). Monitor per-model cost via tags. Likely 50%+ savings."

**Scenario 3: "GPUs are idle 60% of time in training — why, fix?"**
> "GPU starvation — GPUs data ka wait kar rahe (data loading bottleneck) ya small batch (under-fed) ya slow interconnect (communication wait in multi-GPU). Fix: efficient data loading (prefetch, parallel loaders, data near GPU), larger batch, faster interconnect (NVLink), check NCCL. Profile to find exact cause. Idle GPU = wasted money."

**Scenario 4: "Serve model to spiky traffic cost-effectively."**
> "Spiky → autoscale + scale-to-zero (SageMaker Serverless or K8s KServe scale-to-zero) — idle pe no GPU cost, spike pe scale up (cold-start trade-off). Or MIG-shared GPU for baseline + burst. Caching for repeated. Quantize for smaller/cheaper GPU. Right-size."

---

## 4. Tera Edge — GPU me Networking/Infra Connect

Interview me yeh angles se tu shine karega (tera 15-saal):
- **Interconnect** = tera bandwidth/topology domain (NVLink/InfiniBand — GPUs ke beech ka "network")
- **Utilization** = resource efficiency (tu links optimize karta, ab GPUs)
- **Cost** = FinOps (tu AWS cost jaanta)
- **MIG/sharing** = virtualization/partitioning (VLANs jaisa)
- **Cluster networking** = distributed training ka inter-node = tera network fabric

**Interview positioning:** "GPU infra me mera networking background directly value karta — inter-GPU/inter-node communication (NVLink/InfiniBand) essentially high-performance networking hai, aur utilization/cost optimization mera FinOps + resource-efficiency mindset. GPU compute naya hai par infra principles wahi."

---

## Tera GPU Doc DONE!

4 files — fundamentals (VRAM/CUDA/types) → training/inference (multi-GPU/interconnect/utilization) → cloud GPU/cost (AWS/spot/MIG) → interview mastery. Tera weakest area ab covered.

**Revision:** GP-01 (fundamentals — VRAM math), GP-02 (multi-GPU/interconnect — tera networking connect), GP-03 (cost — tera FinOps), GP-04 (scenarios).

**Interview power:** Memory math (params × bytes) ratta — yeh quick calculation impress karta. Aur GPU ko apne networking/infra edge se connect kar (interconnect = networking, cost = FinOps).

---

## 🎓 Mock Drill Debrief (Teacher Session — gaps + corrected answers)

> Live mock drill ke results. Concept knowledge solid (~75%), par retrieval + question-discipline kachcha. Yeh 3 gaps interview me khaate — consciously theek karo.

### 3 ASLI GAPS (yaad rakh)
1. **Question-type sunna** — "cost" / "diagnose" wale sawaal ko "troubleshoot/redesign" me convert mat karo. Pehle 2 sec socho: **yeh sawaal kis type ka — cost? design? diagnose? sizing?** Phir usi lane me jawab.
2. **Diagnose vs Fix separate** — ops sawaal me interviewer aksar "kaise pata karoge (diagnose)" maangta. Seedha fix mat do. Structure: pehle "yeh check karunga," phir "phir yeh fix."
3. **4x model/weights pe, GPU capacity pe NAHI** — training 4x hamesha model size pe (60GB→240GB), GPU size (80) pe nahi.

### Corrected Scenario Answers

**Q: 30B model — inference VRAM, single 80GB fit, training GPUs?**
> "(a) 30B FP16 = 60GB. (b) 80GB H100 me weights fit par + activations/KV cache tight (~75-85GB) — 2 GPU ya quantize safe. (c) Training = 60 × 4 = **240GB** (NOT 80×4), = 3 GPU minimum, practically 4-6 with activations/overhead + NVLink node."

**Q: Inference endpoint cost zyada — 3 levers (no troubleshooting)?**
> "Batching + right-size/quantize + autoscale/scale-to-zero. (Bonus: MIG for small models, Savings Plan if steady, caching for repeated queries.)"

**Q: 175B model inference deploy (ek GPU me fit nahi)?**
> "175B FP16 = 350GB. **Tensor parallelism** (fit nahi → todo; data parallel NAHI kyunki copy banega hi nahi). Split across ~5+ × 80GB (ek 8×H100 node). Intra-node **NVLink/NVSwitch** zaroori (tensor = high comm). Cost: INT8 quantize (350→175GB, kam GPU), steady pe p5 reserved/Savings Plan."

**Q: Distributed training slow, 40% util — DIAGNOSE (structured)?**
> "Diagnose pehle: (1) nvidia-smi har node — util pattern; (2) bottleneck isolate — data loading (starvation)? inter-node communication (InfiniBand health/RDMA/congestion — multi-node me prime suspect)? batch size?; (3) profiler se confirm kahan time. **Phir** fix: data→prefetch/parallel loaders, comm→high-comm GPUs same node (NVLink)+NCCL topology-aware, batch→increase."

### Question-type → lane (cheat)
- **"Cost zyada" (inference)** → batching + right-size/quantize + autoscale/scale-to-zero (+MIG, Savings Plan, caching)
- **"GPU idle/slow"** → DIAGNOSE (data loading, batch size, interconnect) → phir fix
- **"Batch/training cost"** → spot + checkpoint
- **"Model bada, deploy"** → fit? tensor parallel + NVLink; VRAM math pehle

### Positioning (tera edge — interview me bol)
> "GPU infra me mera networking background directly value karta — inter-GPU/inter-node communication (NVLink/InfiniBand) essentially high-performance networking hai; utilization/cost optimization mera FinOps + resource-efficiency mindset. GPU compute naya hai par infra principles wahi (topology, bandwidth, oversubscription, capacity planning, cost attribution)."
