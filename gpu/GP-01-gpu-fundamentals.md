# GP-01: GPU Fundamentals (Zero to Expert)

> Bilkul zero se — GPU kya hai, CPU se kyun alag, memory, CUDA, aur types. Foundation pakka to baaki easy.

---

## 1. GPU Kya Hai — Aur CPU Se Kyun Alag

CPU (Central Processing Unit) tera normal processor — kuch powerful cores (4-64), har core bahut smart aur fast, sequential tasks ke liye optimized. Ek core ek time pe ek complex kaam bahut jaldi karta. GPU (Graphics Processing Unit) bilkul ulta — hazaaron chhote, simple cores (thousands of CUDA cores), har core individually slow par saath me **massively parallel** kaam karte. Iska matlab GPU un tasks me bahut fast hai jahan **same operation bahut saare data pe ek saath** karna ho.

ML/deep learning me yehi hota — model training aur inference basically **matrix multiplications** hain (bade-bade numbers ke arrays ka multiplication). Yeh operations highly parallel hain — hazaaron multiplications ek saath ho sakte. GPU isme CPU se 10-100x fast. Isiliye AI GPU pe chalta. CPU ek chef jaisa (ek expert, ek dish achhe se banata), GPU 1000 line-cooks jaisa (har simple, par 1000 same dishes ek saath banate).

**Networking analogy:** CPU = ek high-end router (powerful, complex routing decisions, sequential). GPU = ek massive switch fabric (thousands of simple ports, parallel forwarding). Bulk parallel work me fabric jeetta.

**Interview line:** "CPU few powerful cores (sequential, complex tasks). GPU thousands of simple cores (massively parallel). ML = matrix multiplications (highly parallel) → GPU 10-100x faster. Isliye AI GPU pe."

---

## 2. GPU Memory (VRAM) — Sabse Important Concept

GPU ki apni dedicated memory hoti hai — **VRAM** (Video RAM, e.g., 16GB/40GB/80GB). Yeh AI infra ka **sabse critical constraint** hai. Kyun? Model ko **VRAM me fit hona chahiye** chalane ke liye — model ke weights, activations, aur (LLM me) KV cache sab VRAM me. Agar model VRAM se bada hai → ek GPU me nahi chalega → multiple GPUs chahiye ya quantization.

**Memory math (interview me aata):** Ek model ka VRAM footprint roughly = parameters × bytes-per-param. FP32 = 4 bytes, FP16 = 2 bytes, INT8 = 1 byte. Toh 7B model in FP16 = 7 billion × 2 = ~14GB. 70B FP16 = ~140GB (ek 80GB GPU me nahi → 2 GPUs ya quantize). Training me aur zyada chahiye (gradients, optimizer states — roughly 4x inference). Yeh calculation architect ko aani chahiye.

**Interview line:** "VRAM = GPU dedicated memory, AI ka critical constraint. Model VRAM me fit hona chahiye. Footprint ≈ params × bytes (FP16=2, INT8=1). 7B FP16=14GB, 70B=140GB. Training ~4x (gradients+optimizer). Iske hisaab se GPU/count decide."

---

## 3. CUDA — GPU ka Software Layer

**CUDA** (Compute Unified Device Architecture) NVIDIA ka platform/API hai jisse software GPU ko use karta. Tu directly CUDA nahi likhega — PyTorch/TensorFlow internally CUDA use karte GPU pe compute karne ko. Important awareness: **cuDNN** (deep learning primitives), **NCCL** (multi-GPU communication library — distributed training me critical), **CUDA version compatibility** (driver + CUDA + framework versions match hone chahiye — common production issue). NVIDIA ka CUDA ecosystem hi reason hai ki NVIDIA GPUs AI me dominate karte (software lock-in).

**Networking analogy:** CUDA = GPU ka "OS/driver + SDK" — jaise device ka NOS (IOS/JunOS) hardware ko usable banata. Version compatibility = firmware/software version matching (mismatch = problems, tu jaanta).

**Interview line:** "CUDA = NVIDIA GPU software platform. Frameworks (PyTorch/TF) isse GPU use karte. cuDNN (DL primitives), NCCL (multi-GPU comm). Version compatibility (driver+CUDA+framework) critical — common prod issue. CUDA ecosystem = NVIDIA's AI dominance."

---

## 4. GPU Types (AI ke liye)

Awareness ke liye common GPUs: **Consumer** (RTX 4090 — cheap, chhote experiments, 24GB). **Data center (NVIDIA):** **T4** (cheap inference, 16GB), **A10/A10G** (inference, 24GB), **A100** (training + big inference, 40/80GB, workhorse), **H100** (latest, fastest, LLM training, 80GB), **L4/L40** (inference optimized). Bade LLM training = H100/A100 clusters; inference = A10/L4/T4 (cost) ya A100 (big models). AWS naming alag (neeche GP-03). Architect ko rough idea — kaunsa workload kaunsa GPU (cost vs capability).

**Interview line:** "GPUs — T4/A10 (inference, cheaper), A100 (training + big inference, 40/80GB workhorse), H100 (latest, LLM training), L4/L40 (inference-optimized). LLM training = H100/A100 clusters, inference = A10/L4 (cost) or A100 (big). Match workload to GPU."

---

## 5. Precision & Data Types (Quantization se connect)

Numbers GPU me alag precision me store hote: **FP32** (32-bit, full precision, accurate, 4 bytes), **FP16/BF16** (16-bit, half — 2x less memory, faster, standard for training/inference now), **INT8** (8-bit integer, quantized — 4x less than FP32, faster, minor accuracy loss), **INT4** (4-bit, aggressive quantization). Trade-off: lower precision = less memory + faster + cheaper, par accuracy thodi kam. Modern AI FP16/BF16 default, quantization (INT8) for cost-optimized inference. Yeh tera LLM Serving doc ke quantization se directly connect.

**Interview line:** "Precision — FP32 (full, 4B), FP16/BF16 (half, 2B, standard), INT8 (quantized, 1B, minor accuracy loss), INT4 (aggressive). Lower = less memory+faster+cheaper, accuracy trade-off. FP16 default, INT8 for cost inference."

---

## Interview Q&A (Fundamentals)

**Q: CPU vs GPU AI ke liye?** — "CPU few powerful cores (sequential), GPU thousands simple cores (parallel). ML = parallel matrix ops → GPU 10-100x faster."

**Q: VRAM kyun important?** — "Model VRAM me fit hona chahiye. Footprint = params × bytes (FP16=2). 70B=140GB → multi-GPU/quantize. Critical constraint."

**Q: CUDA kya?** — "NVIDIA GPU software platform, frameworks isse GPU use karte. NCCL (multi-GPU), version compatibility critical."

**Q: 13B model kitna VRAM?** — "13B × 2 bytes (FP16) = ~26GB inference. Ek 40GB A100 me fit. Training ~4x = ~100GB+ (multi-GPU)."

**Q: FP16 vs INT8?** — "FP16 = 2 bytes, standard. INT8 = 1 byte, quantized, 2x less memory + faster, minor accuracy loss. Cost inference me INT8."

---

## 🎓 Deep Dive & Q&A (Teacher Session — extra clarity)

> Yeh section un sawaalon/confusions ka nichod hai jo detailed teaching me clear hue. Doc ke short points ka "why-level" expansion — revision ke liye gold.

### CPU vs GPU — asli why (sirf "parallel" nahi)
- CPU = **latency-optimized** (ek kaam jitni jaldi ho khatam karo). GPU = **throughput-optimized** (total work-per-second max, chahe ek operation slow ho).
- GPU ka doosra hidden weapon = **memory bandwidth** (A100 ~2 TB/s vs CPU RAM ~50-100 GB/s). "GPU fast" ka aadha reason cores, aadha bandwidth.
- CPU parallelism = **task parallelism** (few cores, alag-alag kaam). GPU = **data parallelism / SIMD** (ek operation, hazaaron data points, ek saath).
- **GPU weakness:** (1) sequential kaam (step 2 ko step 1 ka result chahiye — wait), (2) branchy/decision-heavy (if-else). Yeh CPU better. Example: DB query, OS, web server (sequential + branchy).
- **Rule:** parallel + simple + independent = GPU. Sequential ya branchy = CPU.

### Cores & memory — sharing model
- **Core count per-GPU hota:** ek A100 ~7,000 CUDA cores. 8-GPU node = ~56,000 cores BUT **8 alag memory islands** (har GPU ki apni VRAM).
- **Ek GPU ke andar** cores usi GPU ki VRAM (shared memory) se coordinate karte — CPU jaisa.
- **Alag GPUs ke beech** shared memory NAHI — har GPU alag VRAM island. Data explicitly **NVLink/PCIe pe copy** hota (message-passing, shared memory nahi).
- **CUDA cores** (general parallel workers) vs **Tensor cores** (matrix-multiply specialists, DL ke asli hero).
- Multi-GPU speedup interconnect pe depend — heavy inter-worker dependency = communication-bound = **sublinear** (8 GPU ≠ 8x; kabhi 2-3x).

### Model = numbers, CODE NAHI (badi clarity)
- Model ek software/app **nahi** hai. Model = **numbers ki ek badi file** (jaise Excel with arabon cells), sirf data.
- "7B model" = us bade formula me **7 arab numbers (parameters/weights)** bhare hain.
- Formula soch: `output = (input × A) + B` — yahan A, B = parameters. Asli model me arabon aise numbers.
- Yeh numbers **training** se aate (random se shuru, examples dekh ke adjust hote jab tak sahi na ho).
- **"100GB model" = 100GB sirf numbers** (code/dependencies nahi). Code chhota (MB), CPU pe; numbers VRAM me.

### VRAM me kya rehta (kitchen analogy)
- **Weights** = recipe book (fixed gyaan, sabse bada hissa).
- **Activations** = beech ke jhoothe bartan (temporary, aate-jaate).
- **KV cache** = LLM ke likhe words ka running note (badhta jaata) — SERVING-time only.

### Precision — number ki quality
- FP32=4B (full detail), FP16/BF16=2B (half), INT8=1B (quantized), INT4=0.5B (aggressive).
- Kam bytes = kam VRAM + faster + sasta, PAR accuracy thodi kam.
- **BF16 vs FP16 (dono 2B):** BF16 ka **range bada** (FP32 jitna) → training-stable, overflow/NaN nahi. FP16 ka precision zyada par range chhota → training me overflow risk.
  - **Training default = BF16** (stability). **Inference = FP16/BF16.** Cost inference = INT8.
- **Quantize** = numbers ko lower precision me convert (FP16→INT8), VRAM aadha, minor accuracy loss. Model ka "compression."
  - PTQ (post-training, fast) vs QAT (quantization-aware training, better accuracy, costly).

### Q&A jo session me hue
- **Q: 13B FP16 kitna VRAM, 24GB GPU me fit?** → 13×2 = 26GB. 24GB me **fit nahi** (2GB zyada). Options: INT8 quantize (13GB, fit + sasta — pehla choice) ya 2 GPU (mehenga + comm overhead — jab accuracy critical).
- **Q: KV cache training ka hai ya serving ka?** → **Serving/inference** (text generation). Training ka NAHI. Training memory = weights+gradient+optimizer; serving memory = weights+KV cache.
- **Q: KV cache full — doosre GPU me daal sakte?** → Seedha nahi (alag VRAM islands). Agar model tensor-parallel split hai toh cache bhi split. Warna scale-out (replicas + load balancer) ya PagedAttention. Dedicated offload possible par slow (last resort).

### Interview reflexes (ratta)
- **Inference VRAM** = params × bytes **+ 20-40% extra** (activations + KV cache). GPU thoda bada lo, tight nahi.
- **Training VRAM** = params × bytes **× 4** (weights + gradient + optimizer). 4x hamesha **model/weights pe**, GPU capacity pe nahi.
- 80GB model: inference ~1 GPU (tight), training ~4 GPU minimum, practically 6-8 (activations + overhead + safety margin).
