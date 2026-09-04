# AISD-05: GPU Nodes (VRAM, Parallelism, Interconnect)

> Station 5 — Serving/training ka dil. 3 parts: 5a VRAM, 5b Parallelism, 5c Interconnect.
> (Rahul ka naya area tha GPU — depth se kiya.)

═══════════════════════════════════════════════════════
## 5a — GPU Node + VRAM
═══════════════════════════════════════════════════════

### Paragraph
Har GPU node (p4d.24xlarge) ek EC2 server, andar 4 A100 (80GB each). GPU ki apni memory = VRAM (system RAM se alag). AI ka sabse critical constraint — model weights VRAM me fit hone chahiye. Fit nahi → multi-GPU ya quantize. Architect ko VRAM math aana chahiye.

### Points
1. VRAM = GPU dedicated memory. Weights + KV cache + activations. Fit nahi → single GPU pe nahi chalega.
2. Math: `VRAM ≈ params × bytes`. FP32=4, FP16/BF16=2, INT8=1, INT4=0.5.
   - 7B FP16 = 14GB | 13B = 26GB | 70B = 140GB (→ multi-GPU/quantize)
3. Training ~4x inference (weights + gradients + optimizer states + activations).

### Real footprint nuance
params×bytes = sirf weights. Real ≈ weights × 1.2-2 (KV cache + activations). KV cache concurrency ke saath badhta.

### Gradients + Optimizer (training 4x kyun)
- **Weights** = model ka gyaan (inference me bas yehi). 13B = 26GB
- **Gradient** = "is weight ko kis direction, kitna badlun" (har weight ka apna ek) = 26GB
- **Optimizer state (Adam)** = "is weight ki badalne ki history" (momentum+variance, 2 per weight) = 52GB
- KEY: values chhoti par COUNT = model jitna (13B weights → 13B gradients → 26B optimizer values). Isliye 4x.
```
Inference: weights only        = 26 GB (1x)
Training:  26+26+52+activations ≈ 104 GB (~4x)
```

### 5a Practiced Q — 13B FP16
(a) 26GB (b) 80GB A100 me aaram se (c) INT8 = 13GB. → 10/10

═══════════════════════════════════════════════════════
## 5b — Multi-GPU Parallelism
═══════════════════════════════════════════════════════

### Paragraph
Ek GPU kaafi nahi 2 wajah: model fit nahi (70B=140GB), ya training slow. Dono alag strategy. Interview distinction: "model bada?" vs "training slow?".

### Points
1. **Data Parallelism** — model FIT hai, speed chahiye. Full model copy har GPU, data split, gradients sync. (DDP)
2. **Tensor/Model Parallelism** — model FIT NAHI. Model khud tukdon me split, ek layer multiple GPU pe. (comm-heavy → fast link)
3. **Pipeline Parallelism** — layers stage me (GPU0=layer1-10, GPU1=11-20). Data assembly-line. Bade LLM = 3D (data+tensor+pipeline, DeepSpeed/Megatron).

### Decision rule
```
Model FIT + speed  → DATA
Model FIT NAHI     → TENSOR + PIPELINE
Sabse bade LLM     → 3D (all three)
```

### 5b Practiced Q
(a) 7B fit, slow, 10 GPU → **Data parallelism** (full copy, chunk, gradient sync). ✅ 10/10
(b) 175B (~350GB) fit nahi → **3D (tensor + pipeline + data)**, NOT just pipeline.
   - ⚠️ Pipeline akela kaafi nahi — single layer bhi ek GPU se badi ho sakti → tensor chahiye (layer split).
   - ⚠️ Training = 4x! 350GB weights × 4 ≈ 1400GB → ~18-20 GPU (5 nahi). Inference hota to 350GB/5 GPU.

⭐ Lines: "Layer khud badi → tensor. Layers baantna → pipeline. Speed → data. Bade LLM = 3D."
"Training VRAM = weights × 4. 175B train ≈ 1400GB, inference ≈ 350GB."

═══════════════════════════════════════════════════════
## 5c — Interconnect (NVLink / InfiniBand) + Comm-Bound
═══════════════════════════════════════════════════════

### Paragraph
Multi-GPU me GPUs aapas me baat karte (gradient sync, layer output). Ye communication FAST honi chahiye — warna GPU dusre ka data wait karta → idle → mehenga GPU paisa waste. Isko comm-bound kehte. Do special links: NVLink (intra-node, ~900 GB/s, bahut fast) aur InfiniBand/EFA (inter-node, RDMA).

### Points
1. **NVLink** — intra-node (ek server ke andar). ~900 GB/s. Tensor parallelism (comm-heavy) ek node ke andar rakho.
2. **InfiniBand / EFA** — inter-node (alag servers/AZ). RDMA. NVLink se slow par normal network se bahut fast.
3. **Comm-bound** — slow link → GPU wait → idle → paisa waste. Fix: fast links + NCCL (multi-GPU comm optimize, all-reduce). Placement: comm-heavy = same node.

### Diagram
```
🖥️ Node A                    🖥️ Node B
 GPU0═GPU1═GPU2═GPU3           GPU0═GPU1═GPU2═GPU3
  └─NVLink ~900 GB/s─┘         └─NVLink─┘
       │                            │
       └── InfiniBand/EFA (RDMA) ───┘
           (inter-node, slower)

Slow link → GPU WAIT → idle → comm-bound
Fix: fast link + NCCL + placement (comm-heavy same node)
```

### Interview one-liner
> "Multi-GPU comm: NVLink (~900 GB/s intra-node), InfiniBand/EFA (inter-node RDMA). Slow link → GPU idle → comm-bound. Tensor (comm-heavy) same node. NCCL optimizes all-reduce."

### 5c Practiced Q — tensor parallelism, 4 GPU same node vs alag nodes?
A: Same node (NVLink). Kyun: tensor comm-heavy → fast link chahiye. Galat chuna (inter-node) → slow link → GPU idle → comm-bound → waste.

⚠️ Units: NVLink = ~900 **GB/s** (bytes), InfiniBand = ~400 **Gb/s** (bits). Interview me units galat = impression kharab.

⭐ Final line: "Tensor → same node (NVLink). Pipeline/data → inter-node chalta (IB/EFA). Comm-heavy + slow link = GPU idle = money waste."

═══════════════════════════════════════════════════════
## STATION 5 COMPLETE ✅
═══════════════════════════════════════════════════════
Scores: 5a 10/10, 5b 8/10 (pipeline akela nahi, 3D chahiye; training=4x), 5c 9.5/10 (GB/s vs Gbps unit).
