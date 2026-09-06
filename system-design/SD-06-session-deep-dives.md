# SD-06: Teacher-Session Deep Dives — Modules 1-7 (Consolidated)

> Ye file is teaching session ke saare deep-dive chapters ek jagah — dobara padhne ke liye.
> Modules covered: M1 Foundations (CAP, DB) | M2 ML Fundamentals | M3 LLM/GenAI |
> M4 Production (HA/DR, FinOps, Bottleneck) | M5 DevOps/K8s/AWS | M7 Interview Skills.
> Har topic: concept + points + diagram + interview one-liner + Q&A seekha + ⚠️ galtiyan pakdi.





> Session me deep padhe topics: 1.4 CAP, 1.8 DB, 2.1-2.6. Har topic: concept + points + diagram + interview one-liner + Q&A seekha + galtiyan.

═══════════════════════════════════════════════════════════
## 1.4 — CAP Theorem
═══════════════════════════════════════════════════════════

### Concept
Distributed system me jab **network partition** ho (2 nodes ke beech link toota), tab choice: **available** raho (jawab do, stale chalega) ya **consistent** raho (galat data mat do, reject karo). Dono ek saath nahi. Mathematical impossibility (Brewer diya, Gilbert-Lynch proved).

### Key insight
- **P (partition) hamesha mandatory** — network kabhi bhi fail hoga, opt-out nahi. "CA system banaunga" = GALAT (biggest misconception).
- Asli choice sirf **CP vs AP**, aur wo bhi **sirf partition ke DAURAN**. Network theek → C+A dono milte.

### 3 letters
- **C** = har read latest write dikhe ya error (linearizability, ACID ka C nahi)
- **A** = har request non-error response (bina latest guarantee)
- **P** = messages drop/delay ke bawajood system chale

### CP vs AP
- **CP** (C rakho, A sacrifice) → partition me reject/block. Ex: banking, DynamoDB strong-consistent read, etcd.
- **AP** (A rakho, C sacrifice) → jawab do, stale, baad me sync. Ex: S3, DNS, DynamoDB default, Cassandra.

### Bank example (2 branch)
Delhi + Mumbai, dono ₹1000, phone line sync rakhti. Line kati:
- CP: "Mumbai se baat nahi, mana kar deta" (A gaya, C bacha)
- AP: "Le lo ₹1000" (A bacha, double-withdraw risk, C gaya)

### Q&A me seekha
Amazon cart = **AP** (availability=revenue, stale cart cheap-to-fix). Checkout/payment = **CP** (money inconsistent nahi ho sakta). Same product, alag component alag choice = senior signal.

### Interview one-liner
> "P is not optional in a distributed system — networks fail. So CAP is really a forced choice between CP and AP only during a partition. When healthy, you get both. Cart is AP (availability drives revenue), checkout is CP (money must be consistent)."

═══════════════════════════════════════════════════════════
## 1.8 — Database Scaling
═══════════════════════════════════════════════════════════

### Concept
Ek register (DB) pe 1000 customer = line. Do raaste: copies banao (replication) ya tukdon me baanto (sharding).

### Replication (read problem)
- 1 primary (likho) + kai replicas (padho). Reads baant gaye = fast. Bonus: primary mara → replica promote = HA.
- AWS: RDS read replicas. Catch: replica thoda peeche (replication lag) = eventual consistency.

### Sharding (write/size problem)
- Data key se alag machines pe (A-M shard1, N-Z shard2). Writes baant gaye, data fit.
- AWS: DynamoDB (partition key se auto). Catch: cross-shard join mushkil + hot partition risk.

### Shard key (Q&A me DEEP drill)
Achhi key: **high cardinality** (bahut unique values) + **even access** (koi ek value baar-baar hit na ho).
- ⚠️ GALTI PAKDI: "country" key → India/US pe load toot = **hot partition**. `user_id` hamesha better (crores unique values, even access).
- Rule: "kya koi ek value pe load toot sakta?" haan (country/date/status) = risk. nahi (user_id/order_id) = safe.
- Order example: `order_id` (high cardinality) ✅ vs `order_status` (3 values → hot partition) ❌.

### SQL vs NoSQL
- SQL (RDS/Aurora): structured, joins, strong consistency (banking, registry). Sharding mushkil.
- NoSQL (DynamoDB): massive scale, simple key access, flexible. Sharding built-in, no joins.

### Interview one-liner
> "Read-heavy → replication (primary + read replicas, minor lag). Write/size-heavy → sharding (data split, cross-shard joins hard). Shard key must be high-cardinality + even-access, else hot partition. SQL for relations/consistency, NoSQL for scale."

═══════════════════════════════════════════════════════════
## 2.1 — ML vs Normal Software
═══════════════════════════════════════════════════════════

### Concept
- Normal SW: rules **code me** (developer likhta), deterministic, same input=same output. Behaviour badalta only if code badle.
- ML: rules **data se seekhi** (model), probabilistic. Behaviour badal sakta **code same ho par data badle** (drift).

### Key: 3 cheezein version karni
Normal = sirf code (Git). ML = **code + data + model** (teeno track). Ek bhi badla = alag behaviour. Yahi MLOps ka core.

### Training vs Inference
- Training: ek baar/periodic, bhaari (GPU, ghante, poora dataset). Model banata.
- Inference: baar-baar, live, halka-fast. Model use karta.

### Q&A me seekha
"Normal app touch only on code change. ML needs continuous care because behaviour depends on data — drifts with time even with same code. Hence monitor + version 3 things (code+data+model)."

### Interview one-liner
> "In normal software developers write the rules (deterministic, version code only). In ML the model learns rules from data, so behaviour can change even when code stays same (drift) — that's why ML versions three things: code, data, model, and splits into heavy training vs light inference."

═══════════════════════════════════════════════════════════
## 2.2 — Training Infra
═══════════════════════════════════════════════════════════

### 3D Parallelism (model > 1 node)
- **Data parallelism** — across replicas (poora model har replica pe, data baant)
- **Tensor parallelism** — within node (ek layer ko GPUs me split)
- **Pipeline parallelism** — across nodes (layers ko nodes me baant)
- 175B = teeno saath (3D), sirf pipeline nahi.

### VRAM math (WEAK SPOT — recovered)
```
Inference ≈ params × 2 bytes (FP16)   → 175B = 350GB
Training  ≈ 4× that (gradients + optimizer states + activations) → ~1400GB
```
⚠️ GALTI PAKDI: "350 × 4" karke double-count mat karo. Inference=350GB, training=1400GB (usme 350 andar). 2 bytes × 4 ≈ 4x, ek hi baar.

### Interconnect
NVLink (intra-node, GPU-to-GPU fast), EFA (inter-node). Checkpointing (spot interrupt se resume), spot instances (sasta).

### Interview one-liner
> "175B in FP16 ≈ 350GB to hold (inference); training needs ~4× (gradients + optimizer + activations) ≈ 1400GB — beyond one node, so 3D parallelism: tensor within-node, pipeline across-nodes, data across-replicas."

═══════════════════════════════════════════════════════════
## 2.3 — Serving Patterns
═══════════════════════════════════════════════════════════

### Decision: "jawab kitni jaldi chahiye?"
- **Online** — real-time API, ms, user waiting (chatbot, fraud check). SageMaker endpoint / EKS vLLM.
- **Batch** — bulk, scheduled, latency-tolerant (nightly all-user recommendations).
- **Streaming** — continuous data (Kafka/Kinesis), live fraud.
- **Serverless (Lambda)** — spiky/kabhi-kabhi, scale-to-zero, PAR cold-start bad for big LLM.

### Q&A me seekha
Nightly all-user recommendations = **batch** (delay OK, bulk, pre-compute store). Batch compute + online serve = common combo.

### Interview one-liner
> "Pattern depends on latency need: online (real-time API), batch (bulk scheduled), streaming (continuous). Serverless for spiky but cold-start hurts big LLMs → keep always-on."

═══════════════════════════════════════════════════════════
## 2.4 — Feature Store ⭐
═══════════════════════════════════════════════════════════

### Concept
Feature = calculated numbers model khaata (avg spend, login count). Do jagah chahiye: training (historical, bulk) + serving (live, ms). Agar calculation alag = **training-serving skew** = model bekaar.

### Feature Store = single source of truth
- **Offline store** (S3/warehouse) — historical, training ke liye
- **Online store** (DynamoDB/Redis) — latest values, serving ke liye, low-latency
- Dono me **same calculation** → skew khatam.

### Skew vs Drift (WEAK SPOT — Q&A me drill)
```
SKEW  = "day-1 se galat" (code/calculation mismatch) → feature store fixes
DRIFT = "baad me galat" (world changed with time) → retrain fixes
```
⚠️ GALTI PAKDI: skew se "drift aa jaata" — NAHI, dono alag. Skew=engineering bug, drift=world change.
Examples: 2020 spam model 2024 fail = DRIFT. dollars-vs-rupees mismatch = SKEW.

### Point-in-time correctness
Training me feature ka us-waqt-ka value lo (aaj ka nahi), warna model future dekhta = **data leakage**.

### Interview one-liner
> "Feature store = single source of truth: offline (S3, training) + online (DynamoDB, serving) with the SAME calculation — that kills training-serving skew (a day-1 code mismatch, different from drift which is world-change-later). Plus point-in-time correctness avoids data leakage."

═══════════════════════════════════════════════════════════
## 2.5 — ML Pipeline
═══════════════════════════════════════════════════════════

### Stages (sahi order)
```
ingest → validate/clean → feature eng → train → evaluate(threshold gate) → deploy → monitor → retrain(drift)
```
LOOP hai, line nahi. Monitoring → drift → retrain trigger (Continuous Training).

### Orchestrator
SageMaker Pipelines / Airflow / Step Functions — stages ko order+schedule+retry ke saath chalata.

### Q&A me seekha
Junior "data lo, train, deploy" — 2 missing: (1) beech ke stages (validate + feature eng — model raw data nahi features pe kaam karta), (2) loop (monitor + drift → retrain). Bonus 3rd: evaluate gate (accuracy threshold pe hi deploy).

### Interview one-liner
> "An ML pipeline is an ordered loop: ingest→validate→feature eng→train→evaluate→deploy→monitor→retrain. Unlike normal software it doesn't end at deploy — monitoring feeds retraining (continuous training). An orchestrator like SageMaker Pipelines runs stages with scheduling, deps, retries."

═══════════════════════════════════════════════════════════
## 2.6 — Deployment Strategies
═══════════════════════════════════════════════════════════

### 4 strategies
- **Blue-green** — instant switch old→new, easy rollback (2 full envs).
- **Canary** — small traffic (5%) to new, risk check, phir badhao.
- **Shadow** — new ko mirror traffic (koi user impact nahi), validate.
- **A/B** — split traffic across models, compare BUSINESS metrics, winner pick.

### Key distinction (Q&A)
**Canary = "naya safe hai?"** (risk check) vs **A/B = "kaunsa business ke liye behtar?"** (compare models on real metrics). Dono traffic split, maksad alag.

### Interview one-liner
> "Blue-green (instant switch, easy rollback), canary (small traffic risk-check), shadow (mirror traffic, no user impact), A/B (split across models, compare business metrics). Canary asks 'is new safe?', A/B asks 'which is better?'"

═══════════════════════════════════════════════════════════
## WEAK SPOTS (Module 1-2)
═══════════════════════════════════════════════════════════
1. Shard key: high cardinality + even access; country=hot partition, user_id=safe
2. Skew (day-1 code mismatch) vs Drift (world change later) — MAT mix
3. Training VRAM = 4× inference; don't double-count 2× then 4×
4. Canary (safe?) vs A/B (which better?)
5. ML pipeline = LOOP not line (retrain), features not raw data



> Ye module AI Infra Architect interview ka DIL hai. Session me deep padha (3.1-3.6). Har topic: concept + points + diagram + interview one-liner + jo Q&A me seekha + galtiyan jo pakdi.

═══════════════════════════════════════════════════════════
## 3.1 — LLM Serving Challenges
═══════════════════════════════════════════════════════════

### Concept
Normal ML model: ek request → ek forward pass → ek jawab. LLM alag: jawab **token-by-token** (autoregressive) banta — pehla token banao, usko wapas daal ke doosra, phir teesra... 500-word answer = model ~500 baar chala (sequential, parallel nahi). Isliye LLM slow + mehnga.

### 3 Challenges
1. **Autoregressive (token-by-token)** — N tokens = N forward passes, sequential. Latency 2 parts: TTFT (time to first token) + per-token speed. Isliye streaming (token bante hi bhejo).
2. **KV cache (memory khau)** — har token ka Key-Value GPU memory me store hota (taaki recompute na ho). Grows per request/turn. **Concurrency = KV cache se bandhi, model size se nahi.**
3. **Cost + low GPU utilization** — GPU mehnga, naive batching me idle (alag requests alag length pe khatam).

### Prefill vs Decode (Q&A me seekha)
- **PREFILL:** poora prompt ek saath (parallel) process → har input token ka K-V cache me store → pehla output token bane. Ye TTFT tak ka time.
- **DECODE:** ek-ek token. Har naya token → sirf apna K-V compute → cache me add → purane cache se reuse. Fast, par cache badhta.

Example "What is the capital of India" (6 tokens):
```
Prefill: [What is the capital of India] → 6 K-V cache me → output "New"
Decode:  "New" → k7 add → "Delhi" | "Delhi" → k8 add → [END]
Cache = input tokens + output tokens DONO (multi-turn me badhta jaata)
```

### GPU VRAM me 3 cheezein (Q&A me clear kiya)
```
1. Model weights → fixed (175B = 350GB), kabhi nahi badalta
2. KV cache     → per request, GROWS (input+output tokens)
3. Activations  → temporary internal compute (har layer), chhota
```
⚠️ GALTI PAKDI: "reply = activations" — NAHI. Reply/output token = final result (bahar). Activations = model ke ANDAR ka temporary compute (intermediate numbers), user ko nahi dikhta.

### Interview one-liner
> "LLM serving is hard for three reasons: autoregressive generation (token-by-token, N sequential passes, slow — split into TTFT + per-token), KV cache grows per request and eats GPU memory (concurrency bound by KV cache not model size), and GPUs are expensive with poor utilization under naive batching. vLLM's PagedAttention and continuous batching fix these."

### Key insight
LLM capacity **tokens me** hoti, requests me nahi. 100 chhoti chat vs 10 lambi chat dono ek GPU bhar sakte. User B (5000 tokens) > User A (50 tokens) memory kyunki zyada tokens = zyada KV cache.

═══════════════════════════════════════════════════════════
## 3.2 — vLLM Internals ⭐
═══════════════════════════════════════════════════════════

### Concept
vLLM = serving engine jo 3.1 ke 2 dard fix karta: memory waste + GPU idle. 2 killer ideas.

### 1. PagedAttention (memory ka ilaaj)
**Purana (problem):** har request ko bada continuous block pehle reserve (jaise 2000 token). Request 100 token me khatam → baaki 1900 WASTE = **internal fragmentation** (60-80% memory waste).

**PagedAttention:** KV cache ko chhote fixed **blocks** (~16 token, configurable) me todo, **on-demand** do (jab zaroorat). No upfront reserve → fragmentation khatam → **2-4x zyada requests** same GPU pe. Bonus: prefix sharing (same system prompt block share).

**OS analogy:** bilkul OS virtual-memory **paging** jaisa — memory pages me, non-contiguous, block table pointer rakhta.

### block_size trade-off (Q&A me seekha)
- Chhota block (8) → kam waste, par zyada blocks = manage overhead.
- Bada block (32) → kam overhead, par last block me thoda waste.
- 16 = achha default. **Allocation memory ki hoti (bytes), par token unit me** (per-token KV size fixed).

### Example: 16-block, 25-token query (Q&A me kiya)
```
Block 1 (16): [tok 1-16] poora
Block 2 (16): [tok 17-25][7 khaali] ← waste sirf last block me (max block_size-1)
Aage token bane → khaali 7 slot bharte jaate
```

### 2. Continuous batching (throughput ka ilaaj)
**Static batching:** batch tab tak ruka jab tak sabse lambi request khatam na ho (slots idle).
**Continuous:** jaise ek request khatam, us slot me turant nayi ghusa do. GPU kabhi idle nahi → **throughput 2-4x** → cost↓.

### Fragmentation (Q&A me seekha)
- **Internal:** reserve 2000, use 100 → 1900 andar band (kisi aur ko nahi de sakte) = waste.
- **External:** khaali tukde bikhre, total kaafi par bada continuous nahi milta.

### Interview one-liner
> "vLLM solves two things. PagedAttention manages KV cache in small fixed blocks like OS paging — no pre-reserved big chunks, so no fragmentation, ~2-4x more concurrent requests, plus prefix sharing. Continuous batching swaps a finished request's slot with a waiting one every iteration, so the GPU never idles — big throughput win that cuts GPU cost."

⚠️ GALTI PAKDI (case study me): "KV cache queries ko GPU se rokta" — NAHI. Semantic cache (Redis, GPU ke BAHAR) queries rokta. KV cache (GPU ke ANDAR) token recompute rokta. PagedAttention = KV cache ko manage karta. Char alag layers — mat mix karo.

═══════════════════════════════════════════════════════════
## 3.3 — Inference Optimization
═══════════════════════════════════════════════════════════

### 3 Levers

**1. Quantization (sabse bada lever)**
Precision ghatao: FP16 (2 bytes/param) → INT8 (1 byte) → INT4 (0.5 byte).
```
175B: FP16=350GB → INT8=175GB → INT4=87GB
```
Faayda: kam VRAM + tez. Keemat: thodi accuracy loss (aksar minimal). Terms: post-training quantization vs quantization-aware training.

**"2 bytes/param" ka matlab (Q&A me seekha):** param = ek number (weight). FP16 = har number 2 bytes me. 175B × 2 = 350GB. Quantization number ki ginti nahi, per-number bytes ghatata.

**2. KV cache optimization**
- Cache ko quantize (FP16→INT8) → aadhi memory.
- Drop old tokens (sliding window, attention sinks).
- GQA/MQA — architecture jo kam KV banata (heads KV share).

**3. Speculative decoding**
Chhota "draft" model kai tokens jaldi guess → bada "target" model ek pass me **verify** → sahi guesses = kai tokens ek pass me. **2-3x faster, quality same** (bada final authority).

**Kyun tez (Q&A me deep):**
```
Chhota model = bade se ~10x TEZ (10ms vs 100ms per token)
Bada verify 5 tokens = 1 PASS (~100ms), NOT 5 passes
Normal: 5 token × 100ms = 500ms
Spec:   chhota 5×10ms=50ms + bada 1 pass 100ms = 150ms → 3x tez
```
⚠️ GALTI PAKDI: "chhota 5 token = 5 sec" — ULTA. Chhota TEZ hota. Aur bada verify 1 pass leta, 5 nahi.
⚠️ Guess **query ke BAAD** live hota (precompute nahi). Guess = next-token prediction.
⚠️ Ye **speed** ke liye hai, accuracy ke liye NAHI (quality bade jitni hi rehti).

### Interview one-liner
> "Three levers: quantization (FP16→INT8→INT4, halves/quarters VRAM + faster, small accuracy cost), KV cache optimization (quantize cache, GQA/MQA, drop old tokens), and speculative decoding (small draft model guesses tokens, big model verifies in one parallel pass for 2-3x speedup, quality unchanged)."

═══════════════════════════════════════════════════════════
## 3.4 — RAG at Scale ⭐
═══════════════════════════════════════════════════════════

### Concept
LLM ko company ka private data pata nahi. Do raaste: retrain (mehnga) ya RAG. RAG = **query time pe relevant docs dhoond ke LLM ko do**, retrain nahi.

### INGESTION (offline, pehle se)
```
1. LOAD + CHUNK → bade docs ko chhote chunks (500 words, OVERLAP ke saath)
2. EMBED → har chunk ka vector (embedding model)
3. STORE → vector DB (Pinecone/OpenSearch/pgvector) + metadata
4. INDEX → HNSW (fast similarity search)
```

**Chunking (sabse critical, Q&A me refine):**
- Chhota chunk → precise, par **context adhoora** (baat tut jaati)
- Bada chunk → poora context, par **noise + kharcha** zyada
- Overlap → baat kate nahi
⚠️ GALTI PAKDI: "code ko chunk" — NAHI, documents/text chunk hote. "Model embedding acche karta" — asli reason RETRIEVAL precise ho (focused chunk = sharp vector = better match).

### QUERY (live)
```
1. EMBED query (SAME model as docs! warna alag vector space)
2. RETRIEVE → top-K (~20) similar chunks (ANN, fast-approximate)
3. RERANK → cross-encoder (slow-accurate) → best 3-5 chuno
4. GENERATE → prompt: "Context: [chunks] + Question" → LLM → answer + citation
```

**Reranking kyun (Q&A):** vector DB fast-but-approximate → 20 laao → reranker accurate-but-slow → best 3-5. Faayda: accuracy↑ + LLM token cost↓ + noise↓. Do-step = best of both.

**Query embedding gotcha (Q&A):** SAME embedding model as docs, warna query aur chunk alag vector space me → similarity meaningless.

### RAG vs Fine-tune
RAG = fresh/changing data, no retrain, citations. Fine-tune = style/domain sikhana. Aksar dono saath.

### Interview one-liner
> "RAG retrieves relevant docs at query time instead of retraining. Ingestion: load→chunk (with overlap)→embed→store in vector DB with HNSW. Query: embed the question with the SAME model, retrieve top-K via fast ANN, rerank with a cross-encoder to pick best 3-5, feed to LLM for a grounded answer with citations. Chunking is the most critical knob; retrieve-then-rerank gives speed + accuracy."

═══════════════════════════════════════════════════════════
## 3.5 — Agents at Scale
═══════════════════════════════════════════════════════════

### Concept
Agent = LLM ko **kaam karwane wala** (sirf jawab nahi). Weather poocha → LLM sochta "API call karu" → tool call → result → final jawab.

**Agent = LLM (brain) + Tools (APIs) + Loop (ReAct)**
- Loop = Reason (socho) → Act (tool call) → Observe (result) → repeat till goal.
- Loop koi alag component nahi, chalne ka PATTERN hai.

### Infra challenges (asli interview point)
1. **Cost explosion** — har loop-step = 1 LLM call. 10-step = 10x cost + latency.
2. **Infinite loop** — stuck ho ke repeat → fix: **max-steps limit** (circuit breaker).
3. **State mgmt** — "ab tak kya hua" rakhna (Redis/DB).
4. **Tool failure** — retry/fallback.
5. **Tracing** — per-step log (kahan galat).
6. **Guardrails** — galat tool call na kare.

⚠️ Don't use agent for simple queries (overkill).

### Interview one-liner
> "An agent = LLM + tools + a ReAct loop (reason→act→observe→repeat till goal). vs a normal LLM's single call, an agent makes many calls, multiplying cost and latency. Key infra concerns: cost explosion, infinite-loop risk (fix with max-steps), state management, tool-failure handling, per-step tracing, and guardrails."

⚠️ GALTI (word): "cost exploitation" → sahi "cost explosion" (cost bhadakna, misuse nahi).

═══════════════════════════════════════════════════════════
## 3.6 — LLM Cost/Latency
═══════════════════════════════════════════════════════════

### Layered strategy (framework)
```
1. CACHE (exact/semantic/prefix) → hit = free, GPU skip
2. ROUTE (small model simple, big complex) = BIGGEST lever (biggest cost driver)
3. COMPRESS prompt + cap output tokens
4. STREAM tokens → perceived latency↓
   (neeche hamesha: quantized models + vLLM)
```

**Sabse bada lever = routing** — zyadatar queries simple par sab bade model pe ja rahi thi. Route se 50-70% bill gir sakta. Plus semantic cache (repeat queries free).

### Cost flow hotspots (Q&A me kiya)
```
LLM inference (GPU) 💰💰💰💰 — sabse bada
Retrieval/rerank + Monitoring 💰💰
Cache hit ~free
Save top-down: cache → route → compress → optimize GPU
Agent = har step ye flow dobara = cost multiply
```

### Interview one-liner
> "Layered: semantic cache first (avoid repeat calls), route simple queries to a small model and only complex ones to the big model (biggest lever), compress prompts and cap output, stream to cut perceived latency — all on quantized models served by vLLM. Don't send every request to the biggest model."

═══════════════════════════════════════════════════════════
## WEAK SPOTS (Module 3) — mock me drill
═══════════════════════════════════════════════════════════
1. Semantic cache (Redis, bahar, rokता query) vs KV cache (GPU andar, recompute rokta) — MAT mix karo
2. Spec decoding = SPEED not accuracy; chhota model TEZ hota; bada verify 1 pass
3. RAG: same embedding model for query + docs; chunk trade-off dono side bolo
4. Reply ≠ activations (activations = internal temp compute)
5. Agent challenges = cost explosion + infinite loop (not "exploitation")



> Session me deep padhe: 4.4 HA/DR, 4.5 FinOps, 4.7 Bottleneck. (4.1/4.2/4.3/4.6 pehle ho chuke — recap syllabus map me.)

═══════════════════════════════════════════════════════════
## 4.4 — HA & DR
═══════════════════════════════════════════════════════════

### HA vs DR (asli farak = failure ka size + scope)
```
HA = chhoti failure (server/AZ down), SAME region, no downtime → "chalte rehna"
     (multi-AZ, auto-failover, redundancy) = PREVENTION
DR = badi disaster (poora REGION down), CROSS-region recovery → "dobara khada karna"
     = RECOVERY
```
⚠️ GALTI PAKDI: RTO/RPO **dono DR ke** metrics hain, HA vs DR ka farak nahi. Pehle main ne ulta joda tha (RTO→HA, RPO→DR) — GALAT.

### RTO & RPO (dono DR ke)
- **RTO** (Recovery Time Objective) = downtime tolerance (kitni der me wapas)
- **RPO** (Recovery Point Objective) = data-loss tolerance (kitna data kho sakte)
- Chhota dono = mehnga. Business criticality decide karti.

### DR Patterns (cost↑ / RTO↓)
```
Backup & Restore → sasta, RTO ghante (restore from backup)
Pilot Light      → DB replica ON, app servers OFF → disaster pe app LAUNCH+scale
                   RTO ~10-30 min (SLOW)
Warm Standby     → poora stack ON par CHHOTA size → disaster pe sirf SCALE-UP
                   RTO ~2-5 min (FAST)
Active-Active    → dono region live, traffic split → RTO ~0 (mehnga)
```

### Pilot Light vs Warm Standby (Q&A me confusion clear)
- **Pilot Light** = "data ready, compute OFF" → app LAUNCH karna padta (slow).
- **Warm Standby** = "sab ready par chhota" → sirf SCALE-UP (fast).
- Yaad: "10s of minutes" (tens=10-30 min, Pilot slow) > "minutes" (2-5 min, Warm fast).
- More pre-running = faster + costlier.

### Active-Active blip (Q&A me seekha)
Failover "zero impact" NAHI — us pal ke active connections DROP (reconnect). New requests fine. Minimize: client retry, connection draining, stateless design.
⚠️ "Zero data loss" ≠ "zero user impact" — brief blip aata.

### AI-specific DR
App failover kaafi nahi — ye bhi replicate karo:
- Model artifacts (S3 cross-region replication)
- Model registry
- Feature store online layer (DynamoDB global tables)
- Vector DB
- **GPU capacity ensure** DR region me (warna failover ho gaya par GPU nahi)

### Interview one-liner
> "HA = staying up through small failures within a region (multi-AZ, auto-failover); DR = recovering from a full-region disaster (cross-region). DR is planned via RTO (downtime tolerance) and RPO (data-loss tolerance) — both DR metrics. Patterns: backup/restore → pilot light → warm standby → active-active, trading cost for recovery speed. For AI, also replicate model artifacts, registry, feature store, vector DB, and ensure GPU capacity in the DR region."

═══════════════════════════════════════════════════════════
## 4.5 — Cost / FinOps
═══════════════════════════════════════════════════════════

### 2 foundational rules
1. **Kill idle** — idle GPU sabse mehnga waste → scale-to-zero, shutdown.
2. **Right-size** — jitni zaroorat utna. Over-provisioned GPU = paisa waste (AI ki common galti).

### Spot instances (Q&A me key distinction)
```
Spot = training/batch pe ✅ — 70% sasta, interrupt OK (checkpoint se resume)
Spot = live serving pe ❌ — reclaim = live request DROP = downtime
       → serving pe on-demand/reserved use karo
```

### Other levers
- Reserved/Savings Plans (steady load discount)
- Tiering (small model simple, big complex)
- Semantic cache (repeat calls bachao)
- vLLM (GPU utilization↑)
- Quantization (chhota GPU)
- Tagging (cost visibility — kaun kitna kharch, warna control impossible)

### Cost flow hotspots (Q&A me kiya)
```
LLM inference (GPU)  💰💰💰💰 — sabse bada line item
Retrieval/rerank + Monitoring 💰💰
Cache hit ~free (GPU skip)
Save top-down: cache → route → compress → optimize GPU
Agent = har step flow dobara = cost multiply
```

### Interview one-liner
> "FinOps: first right-size and kill idle (idle GPU is worst waste), use spot for training/batch (interrupt OK, resume from checkpoint) but on-demand/reserved for serving (spot reclaim drops live requests), tier to smaller models, cache repeats, raise GPU utilization with vLLM, quantize, and tag everything for visibility. GPU is the biggest line item."

═══════════════════════════════════════════════════════════
## 4.7 — Scalability & Bottleneck
═══════════════════════════════════════════════════════════

### Concept
Bottleneck = sabse pehle saturate hone wala resource — poore system ki throughput cap karta (jaise pipe ka sabse patla point). Baaki optimize karna waste jab tak weakest link fix na ho.

### METHOD (measure-first, guess nahi) — WEAK SPOT
```
MEASURE (metrics: utilization, latency breakdown, queue length)
  → IDENTIFY (kaun saturate: CPU/GPU/DB/net 100%?)
  → FIX (scale/cache/optimize)
  → REPEAT (ek fix → agla bottleneck emerge)
```

### KEY diagnostic (Q&A me DEEP drill)
```
GPU 100% + slow  → GPU-power bottleneck → scale/quantize/route
GPU LOW% + slow  → NOT GPU! → batching problem (vLLM) OR upstream (retrieval/rerank/net)
                   → adding GPU is USELESS (naya GPU bhi idle rahega)
```
⚠️ GALTI PAKDI: seedha fixes list mat karo (KV/GPU/cache dekhunga). Pehle MEASURE kaun saturate hai, phir ussi ko fix. Utilization number khud bahut batata: low-util + slow = resource nahi, feeding/upstream problem.
⚠️ Junior "GPU add karo" galat kyunki: agar GPU bottleneck nahi (ya idle due to batching), naya GPU bhi idle = cost up, problem waisi.

### AI common bottleneck
Aksar GPU. Idle? → batching (vLLM). Maxed? → scale/quantize/route. RAG me aksar retrieval/rerank bottleneck.

### Interview one-liner
> "A bottleneck is the first resource to saturate — it caps throughput like the narrowest pipe point. Measure first, not guess: check utilization, latency breakdown, queue length; fix the saturated one; repeat. Key: GPU at 100% + slow = GPU bottleneck (scale/quantize); GPU at low% + slow = NOT the GPU, it's batching or upstream — adding a GPU is useless."

═══════════════════════════════════════════════════════════
## WEAK SPOTS (Module 4)
═══════════════════════════════════════════════════════════
1. RTO + RPO = BOTH DR metrics (not HA vs DR)
2. Pilot Light (app OFF, launch, slow) vs Warm Standby (app small-ON, scale, fast)
3. Active-active: brief blip on in-flight conns; data-loss ≠ user-impact
4. Spot = training ✅ / serving ❌ (reclaim drops live requests)
5. Bottleneck: MEASURE-first; GPU low%+slow = batching/upstream NOT add-GPU



> **Learner:** Rahul (15 yr infra/DevOps background) · **Format:** Hinglish notes + English interview one-liners
> **Goal:** AI Infrastructure Architect interview revision
> **Scope:** 9 sections (5A → 5I) covering containers, K8s, GPU scheduling, autoscaling, compute choices, CI/CD-GitOps, IaC, AWS core services, and Platform/SRE resiliency.

---

## 5A — Containers & Registry

### Concept (Hinglish)
Container ka basic idea simple hai — app + saari dependencies ko ek package me bundle karo taaki har environment (dev, staging, prod) me behaviour consistent rahe. "Mere laptop pe chal raha tha" wali problem khatam. Lekin AI workloads me kahani thodi alag hai. Yahan aap sirf app code bundle nahi karte — aap **model + CUDA + PyTorch + GPU drivers** sab ek saath bundle karte ho, aur inn sabki versions **compatible honi chahiye**, warna model start hi nahi hoga. Doosra bada difference — AI images GBs me hoti hain (normal app images MBs me). Isliye **multi-stage build** use karte ho: ek heavy "build stage" jahan compile/install hota hai, aur ek slim "runtime stage" jo sirf zaroori cheezein rakhta hai. Base image bhi **CUDA base image** honi chahiye (plain python nahi), taaki pull fast ho aur cold-start kam ho.

### Key Points
- **Purpose:** Package app + deps → consistency across environments.
- **AI-specific difference #1:** Bundle karta hai model + CUDA + PyTorch + GPU drivers — **saari versions compatible honi chahiye** ya model start nahi hoga.
- **AI-specific difference #2:** Images **huge (GBs)** hoti hain → use **multi-stage build** (heavy build stage vs slim runtime stage) + **CUDA base image** (plain python nahi) → fast pull, less cold-start.
- **ECR best practices:** pinned tags (`:latest` mat use karo), minimal base image, run as **non-root**, image **scanning** on.
- **Model placement — 2 choices:**
  - **Bake into image** → immutable, reproducible, but image bada.
  - **Load from S3 at runtime** → image chhota, but pull latency add hoti hai.

### ASCII Diagram — Multi-stage build
```
┌─────────────────────┐        ┌───────────────────────┐
│   BUILD STAGE        │        │   RUNTIME STAGE        │
│  (heavy, CUDA base)  │ ─────► │  (slim, only runtime)  │
│  compile, pip install│  copy  │  model + minimal deps  │
│  dev tools           │ artifacts│  non-root, small     │
└─────────────────────┘        └───────────────────────┘
        big                            shipped image
```

### Interview One-liner (English)
> "An AI container image differs from a normal one in two ways: it must bundle a version-compatible stack of model, CUDA, PyTorch and GPU drivers or the model won't start, and it's GBs in size, so we use multi-stage builds on a CUDA base image to keep the runtime slim and cold-start low."

### Gotchas / GALTI PAKDI
- ⚠️ **AI image differs from normal in exactly 2 ways** — (1) model+CUDA+PyTorch+drivers must **all be version-compatible** or it won't start, (2) **huge GBs vs MBs** → needs multi-stage build. Yeh 2 points hi differentiator hain, memorize karo.
- Don't use `:latest` — non-reproducible deployments.
- Baking model = immutable but big; S3 runtime load = small but pull latency. Trade-off dono taraf hai.

---

## 5B — Kubernetes Core

### Concept (Hinglish)
Kubernetes ka do core philosophy hai: **declarative** (aap desired state batao, K8s khud waha pahunchega) aur **self-healing** (reconciliation loop continuously actual state ko desired state se match karta rehta hai). Aap "kaise" nahi batate, sirf "kya chahiye" batate ho. K8s ke do halves hain — **control plane** (dimaag) aur **worker nodes** (haath-pair). EKS pe control plane AWS-managed hota hai, aap sirf worker nodes ki tension lete ho.

### Key Points
**Control Plane (EKS-managed):**
- **API server** = entry point (sab requests yahin aati hain).
- **etcd** = state database — ye ek **CP system** hai (consistency over availability).
- **scheduler** = pod placement decide karta hai (kaunsa pod kaunse node pe).
- **controllers** = reconcile loop chalate hain (desired vs actual).

**Worker Nodes:**
- **kubelet** = node pe pods ko execute karta hai.
- **kube-proxy** = networking rules handle karta hai.
- **containerd** = container runtime.

**Primitives:**
- **Pod** = smallest unit, **ephemeral**, recreate hone pe **IP change** ho jaati hai.
- **Deployment** = replicas manage karta hai + rolling update + self-heal. **Hum Deployment banate hain, Pod directly nahi.**
- **Service** = changing pods ke aage **stable IP/DNS** + load-balancing. **Zaroorat isliye kyunki pod IPs change hoti hain.**
- **Ingress** = HTTP routing → EKS pe **ALB** ban jaata hai.

**Config:**
- **ConfigMap** = non-sensitive config.
- **Secret** = sirf **base64-encoded**, **secure NAHI** hai (decode ho sakta hai) → back it with **AWS Secrets Manager / KMS**.

**Storage:**
- **PVC (claim)** → **PV (actual volume)**.
- **EBS** = single-pod, single-AZ, fast block storage → DB/model ke liye.
- **EFS** = shared, multi-AZ, multi-pod file storage → many inference pods reading the **same model**.
- **StatefulSet** = stateful apps ke liye.

**Networking:**
- Har pod ko IP milta hai; EKS pe **VPC CNI** = pod ko real VPC IP deta hai.
- Ingress → ALB.
- **NetworkPolicy** = pod-to-pod firewall.

**Security:**
- **RBAC** = Role + RoleBinding, least-privilege.
- **EKS IRSA** = IAM role mapped to a service account → pod S3/DynamoDB access karta hai **without hardcoded keys**.

### ASCII Diagram — Cluster overview
```
        CONTROL PLANE (EKS-managed)
   ┌──────────────────────────────────────┐
   │ API server ── etcd(CP) ── scheduler   │
   │        └── controllers (reconcile)    │
   └──────────────────────────────────────┘
                    │
        ┌───────────┴────────────┐
     WORKER NODE               WORKER NODE
   ┌─────────────┐           ┌─────────────┐
   │ kubelet     │           │ kubelet     │
   │ kube-proxy  │           │ kube-proxy  │
   │ containerd  │           │ containerd  │
   │  [Pods]     │           │  [Pods]     │
   └─────────────┘           └─────────────┘

   Deployment → manages → Pods (ephemeral, IP changes)
   Service    → stable IP/DNS in front of pods → LB
   Ingress    → HTTP routing → ALB
```

### Interview One-liner (English)
> "Kubernetes is declarative and self-healing: you declare desired state and a reconciliation loop keeps actual state matching it. We create a Deployment (not a bare Pod) because it manages replicas, rolling updates and self-healing, and we put a Service in front because pod IPs change on recreation and the Service gives a stable endpoint plus load-balancing."

### Gotchas / GALTI PAKDI
- ⚠️ **Service ka real reason = pods ephemeral hain, recreate pe IP change hoti hai** → Service stable endpoint + LB deta hai. Yeh "why Service" ka exact answer hai.
- **Secret is NOT secure** — sirf base64, decode ho jaata hai → KMS / Secrets Manager se back karo.
- **etcd is a CP system** — consistency prioritise karta hai.
- Hum **Deployment banate hain, Pod directly nahi** — self-heal aur rolling update Deployment se aata hai.
- **EBS = single-pod/single-AZ** (DB/model), **EFS = shared/multi-AZ/multi-pod** (many pods same model padhte hain). Ye mat mix karo.

---

## 5C — GPU on Kubernetes (⭐ star topic)

### Concept (Hinglish)
Default me Kubernetes **GPU ke prati blind hai** — usse pata hi nahi ki node pe GPU hai. Yahan aata hai **NVIDIA device plugin** ka role: ye GPUs ko ek **schedulable resource** ke roop me **advertise** karta hai (`nvidia.com/gpu: N`). Important: plugin sirf GPUs ko **expose/advertise** karta hai — **node pool create NAHI karta**. Node pool aap khud banate ho ya Karpenter banata hai. Phir pod apne resource limits me `nvidia.com/gpu: 1` request karta hai. GPU nodes mehnge hote hain (p4d/g5), isliye us node pool pe **taint** lagate ho taaki sirf matching **toleration** wale pods hi wahan land karein — normal pods GPU waste na karein.

### Key Points
- **NVIDIA device plugin** = GPUs ko **advertise** karta hai as `nvidia.com/gpu: N`. Bina iske K8s GPU-blind hai. **Plugin only exposes/advertises — does NOT create node pool.**
- **Pod** apne resource **limits** me `nvidia.com/gpu: 1` request karta hai.
- **GPU node pool** = p4d / g5 instances + **taint** → sirf matching **toleration** wale pods hi land karein → expensive GPU ko waste hone se bachaata hai.
- **GPU sharing** (jab full GPU ek chhote model ke liye overkill ho):
  - **MIG (Multi-Instance GPU)** = hardware-isolated slices (A100/H100 pe).
  - **Time-slicing** = no isolation, simple.
- **vLLM on K8s pattern:** Deployment (gpu request + toleration) → GPU pool pe schedule → Service/Ingress → autoscale via **HPA/KEDA** → model loaded from **EFS/S3**.

### ASCII Diagram — GPU scheduling
```
  You / Karpenter ── creates ──► GPU NODE POOL (p4d/g5) [TAINT]
                                        ▲
  NVIDIA device plugin ── advertises ── │  nvidia.com/gpu: N
  (exposes only, no node pool)          │
                                        │
  Pod: limits{ nvidia.com/gpu: 1 } ─────┘
       + toleration (matches taint) ──► lands on GPU node
```

### Interview One-liner (English)
> "Kubernetes is blind to GPUs until the NVIDIA device plugin advertises them as a schedulable resource like `nvidia.com/gpu`; the plugin only exposes GPUs, it does not create the node pool. Pods request `nvidia.com/gpu: 1`, and we taint the expensive GPU node pool so only pods with a matching toleration land there."

### Gotchas / GALTI PAKDI
- ⚠️ **Device plugin EXPOSES GPUs, it does NOT create node pools.** Node pool aap ya Karpenter banate ho. Yeh sabse common trap hai.
- Taint + toleration ka purpose = mehnge GPU nodes ko normal pods se protect karna.
- MIG = hardware-isolated; time-slicing = no isolation but simple. Isolation chahiye to MIG.

---

## 5D — Autoscaling

### Concept (Hinglish)
Autoscaling do levels pe hoti hai — **pod level** aur **node level**. Pod level pe HPA pod **count** badhata hai, VPA pod **size** badhata hai. Node level pe Cluster Autoscaler predefined node groups me **nodes add** karta hai, jabki Karpenter **on-demand right instance type** launch karta hai (GPU ke liye better). AI-specific twist yahan critical hai: GPU pods ko **GPU-utilization ya request-queue-depth** pe scale karo, **CPU pe NAHI** — kyunki GPU max chal raha hota hai lekin CPU idle dikhta hai, to CPU-based HPA kabhi scale hi nahi karega.

### Key Points
- **POD level:**
  - **HPA** = pod **count** on a metric.
  - **VPA** = pod **size**.
- **NODE level:**
  - **Cluster Autoscaler** = predefined node groups me nodes add karta hai.
  - **Karpenter** = right instance type on-demand launch karta hai → **GPU ke liye better**.
- **KEDA** = event/queue-based scaling (SQS/Kafka depth) + **scale-to-zero**.
- **AI-specific:** GPU pods ko **GPU-utilization ya request-queue-depth** pe scale karo, **NOT CPU** (CPU idle dikhta hai jabki GPU maxed, so CPU-based HPA never scales).
- **GPU cold-start** = new node + big image pull + model load = **minutes** → keep a **buffer** ya latency-sensitive LLMs ke liye **scale-to-zero avoid** karo.

### ASCII Diagram — Two scaling levels
```
   POD LEVEL                    NODE LEVEL
 ┌────────────┐              ┌──────────────────────┐
 │ HPA: count │              │ Cluster Autoscaler   │
 │ VPA: size  │              │  → predefined groups │
 │ KEDA: queue│              │ Karpenter            │
 │  scale-to-0│              │  → right instance    │
 └────────────┘              │    on-demand (GPU)   │
                             └──────────────────────┘

   GPU scaling metric: GPU-util / queue-depth  ✅
                       CPU                      ❌ (idle-looking)
```

### Interview One-liner (English)
> "Autoscaling works at two levels — pods (HPA for count, VPA for size, KEDA for queue-based and scale-to-zero) and nodes (Cluster Autoscaler for predefined groups, Karpenter for on-demand right-sizing, which is better for GPUs). For GPU workloads you must scale on GPU-utilization or queue depth, never CPU, because CPU looks idle while the GPU is maxed."

### Gotchas / GALTI PAKDI
- ⚠️ **Scale GPU pods on GPU-util / queue-depth, NOT CPU.** CPU-based HPA GPU workload pe kabhi scale nahi karega.
- **Karpenter > Cluster Autoscaler for GPU** — right instance type on-demand launch karta hai.
- **GPU cold-start = minutes** (node + image pull + model load) → latency-sensitive LLM ke liye buffer rakho / scale-to-zero avoid karo.

---

## 5E — EKS vs ECS vs Fargate vs Lambda

### Concept (Hinglish)
Yeh ek **control-vs-simplicity spectrum** hai. Ek taraf max control aur portability (EKS), doosri taraf max simplicity aur serverless (Lambda). Sabse bada trap: **Fargate ek launch MODE hai, alag orchestrator nahi** — ye ECS/EKS ke *neeche* chalta hai. Isliye "Fargate vs EKS" wali framing galat hai, kyunki Fargate to EKS/ECS ke andar hi run karta hai.

### Key Points
- **EKS** = managed Kubernetes, **max control**, portable/multi-cloud, rich ecosystem, **complex**, **paid control plane**.
- **ECS** = AWS-proprietary orchestrator, **simple**, AWS-only, **free control plane**.
- **Fargate** = **serverless launch MODE** under ECS/EKS — no node management, pay-per-use. **NOT a separate orchestrator** → so "Fargate vs EKS" is wrong framing (Fargate runs *under* EKS/ECS).
- **Lambda** = event-driven functions, **15-min limit**, cold start.
- Both EKS and ECS control planes are **AWS-managed**.
- **AI/GPU gotcha:** Lambda and Fargate have **NO/limited GPU support** → LLM inference goes on **EKS/ECS backed by EC2 GPU instances** (vLLM, GPU node pool, autoscaling). Lambda/Fargate sirf **CPU glue** ke liye.
- **ECS vs EKS (short):** ECS = AWS's easy proprietary way; EKS = standard Kubernetes, portable but complex.

### ASCII Diagram — Control ↔ Simplicity spectrum
```
  MAX CONTROL / PORTABLE                     MAX SIMPLICITY / SERVERLESS
  ┌──────────┐   ┌──────────┐   ┌───────────────┐   ┌──────────┐
  │  EKS     │   │  ECS     │   │  Fargate       │   │ Lambda   │
  │ managed  │   │ AWS-only │   │ LAUNCH MODE    │   │ 15-min   │
  │ k8s      │   │ simple   │   │ under EKS/ECS  │   │ event    │
  │ paid CP  │   │ free CP  │   │ no nodes       │   │ cold st. │
  └──────────┘   └──────────┘   └───────────────┘   └──────────┘

  GPU / LLM inference ──► EKS/ECS on EC2 GPU  ✅
  Lambda / Fargate     ──► CPU glue only       (no/limited GPU)
```

### Interview One-liner (English)
> "It's a control-versus-simplicity spectrum: EKS gives max control and portability with a paid control plane, ECS is AWS's simpler proprietary orchestrator with a free control plane, Fargate is a serverless launch mode running under ECS/EKS (not a separate orchestrator), and Lambda is for short event-driven functions. Since Lambda and Fargate have no/limited GPU support, LLM inference runs on EKS/ECS backed by EC2 GPU instances."

### Gotchas / GALTI PAKDI
- ⚠️ **Fargate is a launch MODE, not a separate orchestrator** — "Fargate vs EKS" galat framing hai, Fargate EKS/ECS ke *under* chalta hai.
- **Lambda/Fargate = no/limited GPU** → LLM inference EKS/ECS + EC2 GPU pe hi jaata hai; Lambda/Fargate sirf CPU glue.
- Both EKS & ECS control planes AWS-managed hain; difference: EKS paid CP + portable, ECS free CP + AWS-only.

---

## 5F — CI/CD + GitOps

### Concept (Hinglish)
**CI** = on push → build + test. **CD** = tested artifact ko auto-deploy karo. **GitOps** ka core idea: **Git = single source of truth**. Aap declarative desired state Git me rakhte ho → automatically versioning + audit + rollback (`git revert`) mil jaata hai. Purane push-based CD me aap bahar se `kubectl apply` karte the with external credentials — jo kam secure hai. **ArgoCD/Flux** ek **in-cluster agent** hai jo Git se **PULL** karta hai aur continuously cluster ko Git se match karta rehta hai. Pull-based zyada secure hai (credentials cluster ke andar rehte hain) aur **drift auto-correct** karta hai.

### Key Points
- **CI** = on push → build + test.
- **CD** = auto-deploy the tested artifact.
- **GitOps:** Git = single source of truth (declarative desired state → versioning + audit + rollback via `git revert`).
- **ArgoCD / Flux** = in-cluster agent that **PULLS** from Git and continuously reconciles cluster to match Git.
  - vs old **push-based CD** (`kubectl apply` from outside with external credentials).
  - Pull-based = **more secure** (credentials stay in cluster) + **auto-corrects drift**.
- **Helm** = templatize/package K8s manifests (values for env-specific config).
- **Argo Rollouts / Flagger** = automated **canary / blue-green** with metric gates.
- **ML extra — CT (Continuous Training):** drift ya new data → auto retrain → **eval gate** → redeploy.

### ASCII Diagram — Pull-based GitOps
```
   Developer ── push ──► Git repo (source of truth, desired state)
                              │
                     (agent PULLS)
                              ▼
                    ┌──────────────────┐
                    │ ArgoCD / Flux    │  in-cluster
                    │ reconcile loop   │  → auto-corrects drift
                    └──────────────────┘
                              │ applies
                              ▼
                         K8s Cluster

   Rollback = git revert   |   Canary/blue-green = Argo Rollouts/Flagger
```

### Interview One-liner (English)
> "GitOps makes Git the single source of truth for declarative desired state, giving you versioning, audit and rollback via git revert. Tools like ArgoCD or Flux run inside the cluster and pull from Git, continuously reconciling and auto-correcting drift — which is more secure than push-based CD because credentials never leave the cluster."

### Gotchas / GALTI PAKDI
- ⚠️ **Pull-based (ArgoCD/Flux) is more secure than push-based** — credentials cluster ke andar rehte hain, bahar se `kubectl apply` nahi.
- GitOps ka rollback = `git revert` (kyunki Git = desired state).
- **CT (Continuous Training)** = ML-specific extra: drift/new data → retrain → **eval gate** → redeploy. Eval gate skip mat karo.
- Helm = packaging/templating; Argo Rollouts/Flagger = canary/blue-green with metric gates. Alag roles.

---

## 5G — Infrastructure as Code (IaC)

### Concept (Hinglish)
Manual console clicks ki jagah infrastructure ko **code** me define karo → reproducible + version-controlled/reviewable + automated. Teen main tools: **Terraform** (HCL, declarative, multi-cloud, most popular), **CDK** (AWS-only, real programming language Python/TS, compiles to CloudFormation), aur **CloudFormation** (AWS raw YAML/JSON). Ek key concept hai **state file** — ye record hai ki abhi kya deployed hai; tool state aur code ke beech **diff** compute karta hai aur sirf changes apply karta hai. Team me state ko **remote (S3) with locking** rakho. **Drift** tab hoti hai jab koi manually out-of-band infra change karta hai → `terraform plan` detect kar leta hai → code re-apply karke reality wapas sync karo.

### Key Points
- **Why IaC:** reproducible + version-controlled/reviewable + automated (vs manual clicks).
- **Terraform** = HCL, declarative, **multi-cloud**, **most popular**.
- **CDK** = **AWS-only**, real language (Python/TS), compiles to CloudFormation.
- **CloudFormation** = AWS raw YAML/JSON.
- **State file** = record of what is currently deployed → tool computes a **diff**, applies only changes. Team ke liye **state remote in S3 with locking**.
- **Drift** = someone changed infra manually out-of-band → `terraform plan` detects it → re-apply code to sync reality back.
- **Rule:** always run **plan before apply**.
- **Choosing:** multi-cloud/popular → **Terraform**; AWS-only + programming language → **CDK**.

### ASCII Diagram — Plan/Apply + drift
```
   Code (HCL/CDK) ──┐
                    ▼
              ┌───────────┐   diff   ┌──────────────┐
   State file │ terraform │ ───────► │ apply changes│
   (S3+lock) ─┤   plan    │          │ only         │
              └───────────┘          └──────────────┘
                    ▲
   Drift: manual out-of-band change ─┘  (plan detects → re-apply)
```

### Interview One-liner (English)
> "IaC replaces manual console clicks with version-controlled, reviewable, reproducible infrastructure. The state file records what's currently deployed so the tool diffs code against it and applies only the changes; drift — an out-of-band manual change — is caught by terraform plan, which is why you always plan before apply. Use Terraform for multi-cloud and CDK when you're AWS-only and want a real programming language."

### Gotchas / GALTI PAKDI
- ⚠️ **Always `plan` before `apply`** — diff dekhe bina apply mat karo.
- **State remote in S3 with locking** for teams — warna concurrent applies corrupt kar denge.
- **Drift** = out-of-band manual change; `terraform plan` isko detect karta hai, phir code re-apply karke sync karo.
- Tool choice rule: multi-cloud/popular → Terraform; AWS-only + real language → CDK.

---

## 5H — AWS Core Services (by category, AI defaults)

### Concept (Hinglish)
Interview me AWS services ko **category-wise** yaad rakho, aur har category me AI ka **default choice** pata ho. Neeche category-by-category breakdown hai. Sabse important AI-specific mappings: model artifacts → S3, online feature store → DynamoDB, vector DB for RAG → OpenSearch, pod→AWS access without keys → IRSA.

### Key Points — by category
**Compute:** EC2 GPU (**p4d/p5/g5**), Fargate, Lambda.

**Networking:** VPC, **ALB** (L7 HTTP), CloudFront (CDN), Route53 (DNS, failover routing), PrivateLink (private service access).

**Storage:**
- **S3** = model artifacts / datasets / **offline feature store**.
- **EBS** = single-pod block.
- **EFS** = shared file.
- **FSx** = high-perf training datasets.

**Database:**
- **RDS/Aurora** = model registry / metadata (SQL).
- **DynamoDB** = **online feature store** (NoSQL).
- **ElastiCache-Redis** = cache / **semantic cache**.
- **OpenSearch** = **vector DB for RAG**.

**Messaging:**
- **SQS** = batch inference jobs.
- **SNS** = pub-sub.
- **Kinesis** = streaming.
- **EventBridge** = event routing.

**Security:**
- **IAM** (**IRSA** = pod→AWS access without keys).
- **KMS** = encryption keys.
- **Secrets Manager** = creds/API keys.
- **WAF** = web firewall.
- **Cognito** = user auth.

**Observability:**
- **CloudWatch** = metrics/logs/alarms.
- **X-Ray** = distributed tracing.
- **Container Insights** = EKS/ECS metrics.

### ASCII Diagram — AI defaults cheat-map
```
 Compute   : EC2 GPU (p4d/p5/g5) | Fargate | Lambda
 Network   : VPC | ALB(L7) | CloudFront(CDN) | Route53 | PrivateLink
 Storage   : S3(models/datasets/offline FS) | EBS | EFS | FSx(train)
 Database  : RDS/Aurora(registry) | DynamoDB(online FS)
             ElastiCache-Redis(semantic cache) | OpenSearch(vector/RAG)
 Messaging : SQS(batch) | SNS(pubsub) | Kinesis(stream) | EventBridge
 Security  : IAM/IRSA | KMS | Secrets Mgr | WAF | Cognito
 Observe   : CloudWatch | X-Ray(trace) | Container Insights
```

### Interview One-liner (English)
> "For AI defaults on AWS: S3 for model artifacts, datasets and the offline feature store; DynamoDB for the online feature store; OpenSearch as the vector DB for RAG; ElastiCache-Redis for semantic caching; EC2 p4d/p5/g5 for GPU compute; and IRSA to let pods access AWS services without hardcoded keys."

### Gotchas / GALTI PAKDI
- ⚠️ **IRSA vs S3 Gateway Endpoint are DIFFERENT things:**
  - **IRSA** = permission/identity (lets a pod access S3 *without keys*).
  - **S3 Gateway Endpoint** = private network path (routing).
  - Ye do alag layers hain — permission ≠ network path. Interview me confuse mat karna.
- **Offline feature store = S3**, **online feature store = DynamoDB** — dono alag.
- **OpenSearch = vector DB for RAG**; **Redis = semantic cache**. Alag roles.

---

## 5I — Platform / SRE (Resiliency)

### Concept (Hinglish)
SRE ki core mentality: **failure expected hai, exception nahi.** Design aise karo ki failure **isolate** ho, **cascade** na kare. Yahan kuch distinct patterns hain jinhe **confuse nahi karna** — sabka purpose alag hai. Retry (transient failures ke liye), Circuit breaker (cascade rokne ke liye), Graceful degradation (kuch lower-quality karo), Load shedding (kuch requests reject karo). Sabse common trap: **degradation (kuch karo) vs load shedding (reject karo)** ko mix kar dena.

### Key Points — distinct patterns (do NOT confuse)
- **Retry with exponential backoff + jitter** = transient failures ke liye; failing service ko flood mat karo (jitter isliye).
- **Circuit breaker** = dependency N baar fail → **OPEN** = calling band → wait → **half-open** test → **close**. Cascade failure + retry-storm ko rokta hai.
- **Graceful degradation** = still do **SOMETHING** lower-quality (cached answer serve karo ya chhote model pe route karo).
- **Load shedding** = **REJECT** some requests (503) under overload to protect the rest. **This is NOT degradation** (rejecting ≠ degrading).
- **Timeout** = every external call pe timeout lagao.
- **Bulkhead** = isolate workloads (e.g., separate **training vs inference node pools**).
- **Capacity planning** = proactive.
- **AI:** agar LLM down/slow hai → fallback to **cache ya smaller model**, crash mat karo.

### ASCII Diagram — Circuit breaker states
```
        failures ≥ N
  CLOSED ─────────────► OPEN ──(wait)──► HALF-OPEN
    ▲                    │                 │
    │  (test succeeds)   │  stop calling   │ test call
    └────────────────────┴─────────────────┘
                        (fail → back to OPEN)

  Degradation = serve cached / smaller model   (do something)
  Load shedding = reject with 503              (reject, protect rest)
```

### Interview One-liner (English)
> "In SRE, failure is expected, so you isolate rather than cascade. Retry with backoff and jitter handles transient errors without flooding; a circuit breaker opens after repeated failures to stop the retry storm and prevent cascade; graceful degradation still serves something lower-quality like a cached answer or a smaller model; and load shedding rejects some requests with 503 to protect the rest — which is not degradation, because rejecting is not the same as degrading."

### Gotchas / GALTI PAKDI
- ⚠️ **Cached answer = graceful degradation; 503 reject = load shedding.** These are DIFFERENT patterns — rejecting ≠ degrading.
- **Circuit breaker** ka main purpose = cascade failure + retry-storm rokna (OPEN → half-open → close cycle).
- **Retry me jitter** zaroori hai — warna sab clients ek saath retry karke failing service ko aur flood kar denge.
- **Bulkhead** = training vs inference node pools alag rakho, taaki ek doosre ko na dubaaye.
- AI fallback = cache ya smaller model, **crash nahi**.

---

# WEAK SPOTS (Module 5)

High-priority traps to nail before the interview — ye woh points hain jahan galti sabse aasaan hai:

1. **Device plugin EXPOSES GPUs, does NOT create the node pool** (node pool = you / Karpenter). *(5C)*
2. **Service exists because pod IPs change** on recreate → Service gives a stable endpoint + load-balancing. *(5B)*
3. **IRSA (permission/identity) vs S3 Gateway Endpoint (network path)** — different layers; permission ≠ routing. *(5H)*
4. **Graceful degradation (do SOMETHING lower-quality) vs Load shedding (REJECT with 503)** — rejecting ≠ degrading. *(5I)*
5. **Scale GPU pods on GPU-utilization / queue-depth, NOT CPU** — CPU looks idle while GPU is maxed, so CPU-based HPA never scales. *(5D)*
6. **Fargate is a launch MODE, not a separate orchestrator** — it runs under EKS/ECS, so "Fargate vs EKS" is the wrong framing. *(5E)*

---
*End of Module 5 — DevOps / Kubernetes / AWS revision notes.*



> Session me deep padhe: 7.1 Framework, 7.2 Estimation, 7.3 Trade-off, 7.4 Structure. (7.5 Mocks pending.) Ye "knowledge ko interview me DELIVER karna" sikhata — sabse high-ROI.

═══════════════════════════════════════════════════════════
## 7.1 — Framework Discipline
═══════════════════════════════════════════════════════════

### Concept
Sabse badi galti: sawaal sunte hi seedha solution bolna. Interviewer ko solution nahi, tumhari SOCH KA STRUCTURE dekhna hota. Fixed framework = pilot checklist = na bhoolo na bikhro. Tum interview drive karte ho.

### 5-Step Framework
```
1. REQUIREMENTS CLARIFY (2-3 min) — SKIP MAT KARO
   - Functional: system kya kare? (chat/batch/RAG)
   - Non-functional: QPS, latency, availability/SLA, cost
   - Constraints: users, data, budget
   - POOCHO, maan mat lo. 2-3 clarifying Q = senior signal.

2. ESTIMATION (2-3 min) — numbers do
   - QPS, GPU count, cost. Assumptions LOUD. Round numbers.

3. HIGH-LEVEL DESIGN (5-7 min) — bade boxes + flow
   - Main components + arrows. Poora picture pehle, detail nahi.

4. DEEP DIVE (10 min) — 1-2 components gehrai me
   - Interviewer jise bole ya tum critical chuno. Knowledge yahan chamakta.

5. TRADE-OFFS + BOTTLENECK + FAILURE + WRAP (3-5 min)
   - "X chuna kyunki... alternative Y par..." Bottleneck kahan. Failure kaise.
```

### Q&A me seekha
"LLM chatbot design karo" → pehle REQUIREMENTS. Skip karne se biggest mistake = **galat cheez optimize** (batch kaam ko real-time bana do, ya scale galat maan ke over/under-provision → waste ya crash).

### Interview one-liner
> "I follow five steps: clarify requirements (functional, non-functional, constraints), estimate scale (QPS, GPU, cost with stated assumptions), sketch high-level design, deep-dive one or two critical components, then trade-offs, bottlenecks, failure handling. Starting with requirements — not jumping to the solution — is what separates a senior answer."

═══════════════════════════════════════════════════════════
## 7.2 — Estimation
═══════════════════════════════════════════════════════════

### Method (5 step)
```
1. QPS = users × (requests per user per unit time)
   Ex: 100K users, 1 req/10s → 10,000 QPS
2. Per-unit capacity (ek GPU/server kitna) — assumption loud
   Ex: 1 vLLM GPU ~10 req/s (LLM realistic, NOT 1000!)
3. Count = QPS ÷ capacity
   Ex: 10,000 ÷ 10 = 1,000
4. + Buffer (30% → ×1.3) + HA (multi-AZ spread)
5. Cost = count × $/hr × 730 hr/month → "mehnga? optimize"
```

### AI side-calcs
```
VRAM (inference) = params × 2 bytes → 7B=14GB, 70B=140GB, 175B=350GB
Training VRAM = ×4
Storage = records × size per record
```

### Rules (senior signals)
1. Assumptions LOUD bolo ("maan lo 1 req/10s")
2. Round numbers (10K not 9,847)
3. Buffer + HA hamesha add karo
4. Cost bhi jodo
5. Chup mat ho — assumption bol ke aage

### Q&A me galti + fix (WEAK SPOT)
```
⚠️ Divide carefully — 1666 ÷ 100 = 17 (NOT 166) — decimal slip pakdi
⚠️ Buffer ko NUMBER tak le jao — "30% rakhunga" nahi, "×1.3 = 22" bolo
⚠️ LLM throughput ~10-20 req/s per GPU, NOT 1000 (token-by-token slow)
⚠️ Model size dhyan — 70B = 140GB = multi-GPU, ek me nahi
Practiced: QPS 3000 ÷ 50 = 60 GPU, +30% = 78 ✅
```

### Interview one-liner
> "I estimate top-down with stated assumptions: QPS = users × request-rate, divide by per-unit capacity for the count, add ~30% headroom and multi-AZ for HA, then cost. Round numbers, assumptions out loud — the interviewer wants the approach, not a precise number."

═══════════════════════════════════════════════════════════
## 7.3 — Trade-off Articulation
═══════════════════════════════════════════════════════════

### Concept
Senior vs junior ka sabse bada farak. Koi choice "best" nahi — har choice me gain + cost. Interviewer "aur X kyun nahi?" poochta = test kar raha alternative socha ya nahi.

### Pattern (ratna)
```
"Maine [X] chuna kyunki [requirement/reason].
 Alternative [Y] tha, par [Y ka downside].
 Is case me [X] better kyunki [context]."
= choice + reason + alternative-with-downside
```

### Advanced moves
- Khud apni choice ki KAMZORI bolo ("iska downside ye ki...") — risks jaante ho = senior signal.
- "It depends on [scale/budget/latency]" — context maango, blanket answer mat do.
- STRUCTURED bolo (N reasons + trade-off), ek run-on sentence nahi.

### Common trade-offs (ready rakho)
```
Managed (Bedrock) vs Self-host  → fast/low-ops vs cheap-at-scale/control
CP vs AP                         → consistency vs availability
Spot vs On-demand                → cheap vs reliable
SQL vs NoSQL                     → joins/consistency vs scale
Small vs Big model               → cheap/fast vs accurate
Bake model vs S3-load            → immutable vs small-image
Cache more vs fresh data         → fast/cheap vs staleness
```

### Q&A me seekha (self-host example — strong answer)
"Self-host chuna 3 wajah se: (1) full control, (2) 24×7 heavy load pe Bedrock per-call mehnga (breakeven cross), (3) customization. Trade-off maanta: patching/upgrade/security/sharding meri zimmedari — overhead zyada. Bedrock low/spiky ke liye better, humare high-steady case me self-host jeet-ta."

### Interview one-liner
> "I never call a choice 'best' — every choice has a gain and a cost. Pattern: I chose X because of this requirement; the alternative was Y but it has this downside; in this context X wins. I also state my design's own weaknesses and say 'it depends' — articulating trade-offs, not memorizing one answer, shows seniority."

═══════════════════════════════════════════════════════════
## 7.4 — Structure Discipline
═══════════════════════════════════════════════════════════

### 4 aadatein
1. **"N batao" → exactly N, labeled** (1,2,3) — na kam na zyada na bina number. Poore answer ko number do ("3 parts me todunga").
2. **Thinking out loud** — bol ke socho, chup silence = interviewer ko kuch nahi dikhta.
3. **Time-box** — high-level pehle (poora cover), phir 1-2 deep. Ek jagah atak ke adhoora mat chhodo.
4. **Clean delivery + control** — labeled boxes, structure announce karo, TUM drive karo (react mat karo).

### Q&A me galti (WEAK SPOT)
Structure sahi tha (exactly 3, labeled), par CONTENT galat:
⚠️ "LLM challenges" poocha → maine GOALS diye (fast/cheap/good-UX) instead of technical CHALLENGES (autoregressive, KV cache, cost).
⚠️ Content topic-SPECIFIC rakho — "challenges" = technical pains, NOT goals. Goals ≠ challenges.

### Interview one-liner
> "I keep answers structured: if asked for N things I give exactly N, numbered. I announce structure upfront, think out loud, and time-box — cover the whole design at high level before deep-diving, so I never leave it incomplete. Clean labeled diagrams and driving the conversation reads as senior."

═══════════════════════════════════════════════════════════
## 7.5 — MOCK INTERVIEWS (PENDING — most critical)
═══════════════════════════════════════════════════════════
- 3-5 full mock interviews, main interviewer banunga
- Time pressure, follow-ups, weak-spot drill
- Ye actual "pass/fail" readiness banata
- STATUS: abhi pending — case studies (Module 6) ke baad karenge

═══════════════════════════════════════════════════════════
## BONUS: Capacity/Buffer deep (Q&A me seekha)
═══════════════════════════════════════════════════════════
### 30% buffer fix nahi — reasonable default
```
Steady load → 15-25% | Spiky → 50-100%+ | Critical → zyada | Cost-sensitive → kam
Decide by: traffic pattern + scaling speed (GPU cold-start slow = zyada buffer) + cost-vs-risk
```
### Buffer bhi khatam ho gaya → layered defense
```
1. Autoscaling trigger (par GPU slow)
2. Queue/buffer (SQS absorb)
3. Load shedding (503 reject excess)
4. Graceful degradation (cached/small model)
5. Rate limiting (per-user cap)
→ system DEGRADE ho, CRASH na ho
```

═══════════════════════════════════════════════════════════
## ALL WEAK SPOTS (master list — mock me drill)
═══════════════════════════════════════════════════════════
1. Data drift P(X) vs concept drift P(Y|X)
2. Skew (day-1 code mismatch) vs Drift (world change later)
3. Training VRAM = 4× inference; no double-count
4. 175B = 3D parallelism (not just pipeline)
5. Structure: "N batao" → exactly N labeled; content = specific not goals
6. Bottleneck: MEASURE-first; GPU low%+slow = batching/upstream not add-GPU
7. Semantic cache (Redis, outside) vs KV cache (GPU inside)
8. Spec decoding = speed not accuracy
9. RAG: same embedding model query+docs
10. Estimation: divide carefully, buffer to actual number, LLM ~10-20 req/s not 1000
11. Pattern naming: degradation (do something) vs load shedding (reject 503)
12. IRSA (permission) vs Gateway Endpoint (network path)
13. Device plugin exposes GPU, doesn't create node pool
14. Fargate = launch mode, not separate orchestrator
