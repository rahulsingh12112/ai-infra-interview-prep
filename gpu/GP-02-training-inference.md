# GP-02: GPU for Training & Inference — Multi-GPU, Utilization, Interconnect (Zero to Expert)

> Ab GPU ko training aur inference me kaise use karte, multi-GPU kab/kaise, aur utilization/interconnect (bottlenecks).

---

## 1. GPU in Training vs Inference (alag needs)

Training me GPU pe model seekhta — yeh **memory + compute heavy** hai kyunki model weights + activations + gradients + optimizer states sab VRAM me (roughly 4x model size). Isliye training ko bade/zyada GPUs chahiye. Inference me sirf model weights + activations (+ KV cache for LLM) — kam memory, par **latency-sensitive** (real-time). Toh training = throughput/memory focus (batch, multi-GPU, spot ok), inference = latency + utilization focus (batching, right-size, always-on).

**Interview line:** "Training GPU-heavy — weights+activations+gradients+optimizer (~4x model, big/multi-GPU, spot ok, throughput focus). Inference lighter — weights+activations+KV cache, latency-sensitive, utilization focus."

---

## 2. GPU Utilization — Cost ka Dil

GPU mehenga hai (A100 ~$2-4/hr, H100 more). Isliye **utilization maximize** karna cost ka sabse bada lever — idle GPU = paisa jal raha bina kaam. Utilization measure: `nvidia-smi` (GPU %, memory used). Problems jo utilization girati: chhota batch size (GPU under-fed), data loading bottleneck (GPU data ka wait kare — "GPU starvation"), sequential work. Solutions: **batching** (bade batch = GPU bhar ke chale), **efficient data loading** (prefetch, parallel loaders, data GPU ke paas), **continuous batching** (inference — vLLM). Architect ko utilization monitor karna aur maximize — 30% utilization = 70% paisa waste.

**Networking analogy:** GPU utilization = link utilization — mehenga link idle = waste. Maximize throughput (batching = multiplexing), avoid starvation (buffer/prefetch = QoS). Tu utilization optimize karna jaanta.

**Interview line:** "GPU mehenga — utilization maximize = biggest cost lever. Idle GPU = wasted money. Batching (feed GPU), efficient data loading (avoid starvation), continuous batching (inference). Monitor nvidia-smi. 30% util = 70% waste."

---

## 3. Multi-GPU — Kab Aur Kaise (3 Parallelism types)

Jab ek GPU kaafi nahi (model bada ya training slow), multiple GPUs. Teen strategies:

**Data Parallelism** — same model har GPU pe copy, alag data batches har GPU pe, gradients sync (average). Training speed up (more data parallel). Model ek GPU me fit hona chahiye. PyTorch DDP. Sabse common training scaling.

**Model/Tensor Parallelism** — model khud ek GPU me fit nahi (huge, 70B+), toh model ke layers/tensors ko GPUs me split. Ek layer ka computation multiple GPUs pe (tensor parallel — high communication) ya alag layers alag GPUs pe (pipeline parallel). LLM ke liye zaroori.

**Pipeline Parallelism** — model ki alag stages/layers alag GPUs pe, assembly-line style.

Bade LLM training me **combination** (data + tensor + pipeline = "3D parallelism", DeepSpeed/Megatron). Inference me aksar tensor parallelism (bada model fit karne ko).

**Networking analogy:** Data parallel = load-balancing same work across paths (ECMP). Model/tensor parallel = ek bade computation ko parallel links pe split (needs high inter-link bandwidth). Pipeline = staged processing across nodes.

**Interview line:** "Multi-GPU — data parallelism (model copies, data split, gradient sync — model fits, common training), tensor/model parallelism (huge model split across GPUs — LLM), pipeline (layers staged). Big LLM = 3D combo (DeepSpeed/Megatron). Inference = tensor parallel for fitting."

---

## 4. Interconnect — The Bottleneck (tera networking edge)

Multi-GPU me GPUs ko aapas me baat karni padti (gradient sync, tensor sharing) — yeh communication **bottleneck** ban sakta. Interconnect types (bandwidth order): **PCIe** (standard, slower — ~16-32 GB/s), **NVLink** (NVIDIA's high-speed GPU-to-GPU within a node — ~600-900 GB/s, much faster), **InfiniBand** (across nodes/servers — high-speed cluster networking, RDMA). Bade distributed training me inter-GPU/inter-node bandwidth critical — agar communication slow, GPUs compute ka wait karein (utilization girti). **NCCL** library yeh communication optimize karta. Yeh **tera networking domain** — bandwidth, latency, topology matter.

**Networking analogy:** Yeh literally tera maidan — interconnect = backbone links between processing nodes. NVLink = high-speed backplane (intra-chassis), InfiniBand = high-speed inter-site fabric (RDMA). Bandwidth bottleneck = tu jaanta congestion analysis. Distributed training ki performance interconnect pe utni hi depend karti jitni compute pe.

**Interview line:** "Multi-GPU communication (gradient/tensor sync) = bottleneck. Interconnect: PCIe (slow ~32GB/s), NVLink (intra-node ~900GB/s), InfiniBand (inter-node, RDMA). Slow interconnect → GPUs idle waiting → utilization down. NCCL optimizes. Bandwidth-critical (my networking domain)."

---

## 5. Checkpointing & Fault Tolerance (Training)

Bade model training din-hafte lagta, aur beech me kuch fail ho sakta (GPU error, spot interruption, node crash). Agar shuru se restart karna pade = bahut waste. **Checkpointing** — periodic model state (weights + optimizer) ko save (S3) karna, taaki fail hone pe last checkpoint se resume. Yeh spot instances (interruptible, 70-90% cheaper) use karne ka bhi enabler — interrupt hone pe checkpoint se resume, cost bachao. Frequency trade-off: zyada checkpoint = safe par overhead (save time); kam = fast par zyada loss on failure.

**Networking analogy:** Checkpointing = config backup/snapshot — failure pe last-good se restore, shuru se nahi. Spot + checkpoint = resilient use of interruptible resources.

**Interview line:** "Checkpointing — periodic model+optimizer state to S3, resume on failure (GPU/node/spot interruption) instead of restart. Enables spot instances (70-90% cheaper) safely. Frequency trade-off: safety vs overhead."

---

## Interview Q&A (Training/Inference GPU)

**Q: Training vs inference GPU needs?** — "Training ~4x memory (gradients+optimizer), big/multi-GPU, throughput+spot. Inference lighter, latency-sensitive, utilization focus."

**Q: GPU utilization kyun matter?** — "Mehenga resource — idle = waste. Maximize via batching + efficient data loading. 30% util = 70% cost wasted. Biggest cost lever."

**Q: Multi-GPU strategies?** — "Data parallelism (model copies, data split — common training), tensor/model (huge model split — LLM), pipeline (staged). Big LLM = 3D combo."

**Q: Interconnect kya, kyun matter?** — "GPU-to-GPU communication (gradient/tensor sync). PCIe (slow), NVLink (intra-node fast), InfiniBand (inter-node RDMA). Slow interconnect = GPUs idle waiting. Bandwidth-critical."

**Q: Spot instances safely kaise?** — "Checkpointing — state to S3 periodically, resume on interruption. Spot 70-90% cheaper, checkpoint makes it safe."

---

## 🎓 Deep Dive & Q&A (Teacher Session — extra clarity)

> Detailed teaching ke clarifications ka nichod. Short doc points ka "why-level" expansion.

### Training vs Inference — priority word
- Training priority = **throughput** (bulk kaam jaldi ho) → isliye interruption-tolerant → **spot** chal jaata.
- Inference priority = **latency** (har request fast, user wait kar raha) → stable/always-on → **on-demand/reserved**.
- Networking: bulk file transfer (throughput, resumable) vs real-time voice/video (low latency, stable QoS).

### Utilization — starvation vs congestion (clear karo)
- Utilization girti = **GPU ko kaam/data NAHI mil raha (starvation/underrun)**, na ki zyada mil raha (congestion).
- 2 kaaran: (1) chhota batch → GPU under-fed → fix **batching**; (2) slow data loading → GPU wait (starvation) → fix **prefetch/parallel loaders/data near GPU**.
- Networking: GPU util = link util. Starvation = **buffer underrun** (source slow, pipe khali). Batching = **multiplexing** (pipe bhar ke chalao).

### Multi-GPU — fit-problem vs speed-problem (core distinction)
- Multi-GPU 2 wajah se: **model fit nahi** (todo) ya **kaam slow** (baanto).
- **Data parallelism** = SPEED problem (model ek GPU me fit hai). Har GPU pe **poori model copy**, data split, gradients all-reduce se sync. Low comm (bas gradient sync). ≈ ECMP/load-balancing.
- **Tensor parallelism** = FIT problem (model bada). Ek **layer ko andar se cheer ke** GPUs me (within-layer split). Sab GPU ek saath same layer pe. **High comm → NVLink zaroori.**
- **Pipeline parallelism** = FIT problem (alag tarika). **Alag-alag poori layers alag GPU** pe (across-layer, assembly line). GPU-1 = layer 1-10, GPU-2 = 11-20. Medium comm.
  - **Tensor = horizontal cut (ek layer ke andar). Pipeline = vertical cut (layer-wise stages).** Dono bade/na-fit model ke liye.
- Big LLM = **3D parallelism** (data + tensor + pipeline, DeepSpeed/Megatron). Inference = aksar tensor parallel.
- **Reflex:** Fit nahi = **Tensor**. Fit hai par slow = **Data**.

### All-reduce — asli kyun (real example)
- Data parallel me har GPU **alag data** se **alag sikhta** (GPU-1 billi photos, GPU-2 kutta, GPU-3 cheetah, GPU-4 sher) → 4 copies alag ho jaati.
- Problem: humein ek model chahiye jo **sabkuch** jaane, na ki 4 adhoore.
- **All-reduce = sab GPU ka gradient average karke, wo average sabko wapas** → chaaro copies **same** rehti, milke 4x tez seekhti.
- Bina all-reduce = 4 alag-alag divergent models (bekaar).
- Networking: routers routing updates exchange karke **same routing table** pe aate.

### Interconnect (tera maidan) — 3 levels
- **PCIe** (~16-32 GB/s, slow, general slot) < **NVLink** (~600-900 GB/s, intra-node direct, NVSwitch = non-blocking fabric) < **InfiniBand** (inter-node, RDMA = CPU bypass, low latency).
- Slow interconnect → GPUs compute jaldi khatam, data ka wait → idle → util down → **communication-bound**.
- Design: **high-comm GPUs same node (NVLink), cross-node minimize.** (High-traffic pairs close/fast path — tera network design.)
- **NCCL** = software jo interconnect (hardware) ke **upar** chalta, communication patterns (all-reduce) optimize karta. = routing protocol (BGP) over fiber jaisa. Hardware = NVLink/InfiniBand (carry), NCCL = decide/optimize.

### Checkpointing & spot
- Checkpointing = periodic model+optimizer state → **S3**, failure/interruption pe **resume** (shuru se nahi). = config backup/snapshot, RPO/RTO trade-off.
- **Spot + checkpoint = saste (70-90% off) me safe training.** Frequency ka sweet spot (na zyada = overhead, na kam = failure pe loss).
- Extra safety: spot + kuch on-demand baseline mix.

### Ops discipline (session gap)
- Ops sawaal me: **pehle DIAGNOSE (kya check karunga), phir FIX (kya karunga).** Seedha solution mat do.
- Diagnose funnel: (1) nvidia-smi util pattern, (2) bottleneck isolate — data loading (starvation)? communication (InfiniBand health/RDMA/congestion)? batch size?, (3) profiler se confirm. Phir fix.
