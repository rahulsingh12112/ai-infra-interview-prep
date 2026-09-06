# SD-03: LLM & GenAI System Design (Zero to Expert)

> Yeh 2024-2026 ka hottest topic aur tere role ka core. LLM serving, RAG, agents — sab production scale pe.

---

## 1. LLM Serving — Kya Special Hai

Normal ML model chhota hota (MBs), LLM bahut bada (GBs — 7B model ~14GB, 70B ~140GB in FP16). Isliye LLM serving ke unique challenges: model **VRAM me fit karna** (bade models multiple GPUs pe shard), **latency** (LLM token-by-token generate karta, slow — user ko stream karke dikhana padta), **throughput** (GPU mehenga, maximize utilization via batching), aur **cost** (har token compute, scale pe bill bada). In sab ko solve karne ke liye specialized serving engines aaye jaise **vLLM, TensorRT-LLM, TGI**.

**Networking analogy:** LLM serving = high-bandwidth expensive link pe traffic optimize karna — capacity (VRAM), efficiency (batching = multiplexing), streaming (progressive delivery), cost (metered link).

**Interview line:** "LLM serving unique — huge model (VRAM/multi-GPU sharding), token-by-token latency (streaming), expensive GPU (batching for throughput), per-token cost. Specialized engines: vLLM, TensorRT-LLM, TGI."

---

## 2. vLLM Internals (Interview me impress karta)

vLLM sabse popular open-source LLM serving engine hai, aur iske do key innovations samajhna chahiye. Pehla — **PagedAttention**. LLM generate karte time har token ka "attention KV cache" memory me rakhta, aur yeh memory management inefficient hoti thi (fragmentation, waste). vLLM ne OS ke virtual memory paging idea se inspire hokar KV cache ko "pages" me manage kiya — memory waste dramatically kam, zyada requests ek GPU pe fit. Doosra — **Continuous batching** (ya in-flight batching). Purane systems me batch ke sab requests ke complete hone ka wait hota tha (ek slow request sabko rok deta). vLLM continuously nayi requests ko batch me add/remove karta jaise slots khaali hote — GPU kabhi idle nahi, throughput bahut zyada. In dono se vLLM traditional serving se 10-24x throughput deta.

**Networking analogy:** PagedAttention = memory ko fixed pages me manage (jaise packet buffers/MTU management, fragmentation avoid). Continuous batching = statistical multiplexing — link kabhi idle nahi, jaise-jaise slots khaali, naya traffic bharo.

**Interview line:** "vLLM ke do innovations — PagedAttention (KV cache ko OS-paging jaisa manage, memory waste kam, zyada concurrency) aur continuous batching (requests dynamically add/remove, GPU never idle). Result: 10-24x throughput vs naive serving."

---

## 3. LLM Inference Optimization Techniques

Latency/cost/throughput optimize karne ke techniques (interview me list karna aata ho): **Quantization** — model weights ko lower precision me (FP16 → INT8 → INT4), VRAM aur compute kam, thodi accuracy loss. **Batching** — multiple requests ek saath GPU pe (throughput up). **KV caching** — already-generated tokens ka computation reuse (har token pe poora recompute nahi). **Model parallelism / sharding** — bada model multiple GPUs pe split (tensor parallelism). **Speculative decoding** — chhota fast model draft banata, bada model verify (speedup). **Prompt caching** — repeated prompt prefixes ka cache. **Distillation** — bade model se chhota model train (fast serving). Architect ko trade-offs pata hone chahiye — quantization se speed+cost down par accuracy thodi down.

**Interview line:** "Optimization — quantization (FP16/INT8/INT4, VRAM+cost down, accuracy trade-off), batching (throughput), KV cache (avoid recompute), tensor parallelism (big model multi-GPU), speculative decoding, prompt caching, distillation. Trade-offs: speed/cost vs accuracy."

---

## 4. RAG Architecture at Scale (tera repo se connect)

RAG (Retrieval-Augmented Generation) LLM ko external knowledge deta — LLM ko sab kuch train me nahi, relevant docs retrieve karke context me dete. Production RAG ka pipeline: **Ingestion** (offline — documents load → chunk → embed → vector DB me store) aur **Query time** (online — user query → embed → vector DB me similar chunks retrieve → optionally rerank → prompt me context assemble → LLM → response). System design considerations: **Vector DB choice** (OpenSearch, Pinecone, pgvector — scale/latency/cost), **embedding model** (self-hosted vs Bedrock Titan — latency/cost), **chunking strategy** (size/overlap — retrieval quality), **reranking** (retrieved ko re-order for relevance), **caching** (repeated queries), **latency budget** (embed + retrieve + LLM = total, har step optimize). Scale pe: ingestion pipeline async (queue-based), vector DB sharded/replicated, LLM serving auto-scaled, caching aggressive.

**Networking analogy:** RAG = ek multi-stage lookup pipeline — query aati, indexed store se relevant data fetch (jaise routing table lookup), phir process. Ingestion = building the index (jaise route table populate), query = fast lookup + forward.

**Interview line:** "RAG do phases — offline ingestion (load→chunk→embed→vector DB) aur online query (embed→retrieve→rerank→assemble→LLM). Scale: async ingestion, sharded vector DB, autoscaled LLM, aggressive caching. Latency budget har step optimize."

---

## 5. Agents at Scale

Agents (LLM + tools + reasoning loop — tera LangChain knowledge) production me challenging kyunki har agent request **multiple LLM calls** (reasoning loop) + **tool calls** kar sakta — latency aur cost multiply hote, aur non-deterministic (kitne steps lagenge pata nahi). System design: **loop bounds** (max iterations — infinite loop/cost rokna, tera middleware knowledge), **tool execution** (parallel jahan possible, timeout, error handling), **state management** (conversation/memory — kahan store, Redis), **observability** (har step trace — kya hua, kyun), **cost controls** (per-request budget, model selection — simple pe sasta model). Multi-agent systems me aur complexity — orchestration, communication.

**Networking analogy:** Agent loop = multi-hop routing with decisions at each hop — bound the hops (TTL), handle failures per hop, trace the path. Cost/latency multiply with hops.

**Interview line:** "Agents = multiple LLM+tool calls per request — latency/cost multiply, non-deterministic. Design: max-iteration bounds, parallel tool exec + timeouts, state in Redis, per-step tracing, cost controls (budget + model tiering)."

---

## 6. LLM Cost & Latency Optimization (Architect-critical)

LLM at scale mehenga — cost optimization architect ki badi responsibility. Levers: **Model tiering** (simple queries pe chhota/sasta model, complex pe bada — routing logic), **prompt/response caching** (repeated = no LLM call), **prompt compression** (kam tokens = kam cost), **batching** (throughput), **quantization/self-hosting** (managed API mehenga at scale, self-host on GPU sasta beyond breakeven), **max_tokens limits** (output cap), **semantic caching** (similar queries ka cached response). Latency: streaming (perceived latency down — user turant text dekhta), caching, model size vs speed trade-off, edge/region proximity. Yeh tera FinOps/cost background se connect.

**Interview line:** "LLM cost levers — model tiering (cheap for simple), prompt/response + semantic caching, prompt compression, self-host beyond breakeven, max_tokens caps. Latency — streaming (perceived), caching, region proximity. Continuous cost monitoring."

---

## 7. Bedrock (Tera AWS Stack — Managed GenAI)

AWS role me Bedrock jaanna zaroori. Bedrock = managed foundation models (Claude, Titan, etc.) — API se access, infra manage nahi karna. Key services: **Bedrock (base)** — model invoke, on-demand ya provisioned throughput (guaranteed capacity, cost predictable at scale). **Bedrock Agents** — managed agentic workflows (tools, orchestration — self-manage nahi). **Bedrock Knowledge Bases** — managed RAG (ingestion + vector store + retrieval, tu sirf docs do). **Bedrock Guardrails** — safety/compliance filters (PII, toxicity, topic blocking — tera security angle). Architect decision: managed (Bedrock) vs self-hosted (vLLM on EKS) — managed = fast, less ops, par cost/control trade-off; self-host = cheaper at scale, full control, par ops burden.

**Interview line:** "Bedrock = managed FMs (Claude/Titan), on-demand or provisioned throughput. Agents (managed agentic), Knowledge Bases (managed RAG), Guardrails (safety/PII/compliance). Managed vs self-host (vLLM/EKS) trade-off — ops+cost vs control."

---

## Interview Q&A (LLM/GenAI)

**Q: LLM serving normal ML se kaise alag?** — "Huge model (VRAM/multi-GPU), token-by-token (streaming), expensive GPU (batching), per-token cost. Specialized engines (vLLM)."

**Q: vLLM kyun fast?** — "PagedAttention (KV cache OS-paging jaisa, memory-efficient) + continuous batching (GPU never idle). 10-24x throughput."

**Q: RAG production architecture?** — "Offline ingestion (chunk→embed→vector DB) + online (embed→retrieve→rerank→LLM). Async ingestion, sharded vector DB, autoscaled LLM, caching."

**Q: LLM cost kaise kam?** — "Model tiering, prompt/semantic caching, compression, self-host beyond breakeven, max_tokens caps, monitoring."

**Q: Managed (Bedrock) vs self-host (vLLM)?** — "Managed = fast, less ops, cost/control trade-off. Self-host = cheaper at scale, full control, ops burden. Scale + team + control pe depend."

**Q: Agents scale pe challenge?** — "Multiple LLM+tool calls, latency/cost multiply, non-deterministic. Max-iteration bounds, parallel tools, state mgmt, tracing, cost controls."

---

## 🎓 Deep Dive & Q&A (Teacher Session — zero-se-expert)

> Target-role ka dil. LS/LangChain/RAG-repo se overlap; yahan system-design angle + structure discipline.

### Section 1 — LLM Serving: Kya Special

Normal ML = chhota (MBs), simple. LLM = bada (GBs) + token-by-token. **Challenges (formula: label — kya + kaise):**
1. **Size/Memory** — LLM GBs (7B=14GB, 70B=140GB), ek GPU me fit nahi → multi-GPU sharding (tensor parallel).
2. **Latency/Throughput** — token-by-token slow → **streaming** (perceived latency); mehenga GPU idle na rahe → **batching** (throughput).
3. **Cost** — per-token compute, scale pe bill bada → optimization.
Specialized engines: vLLM, TensorRT-LLM, TGI (normal Flask nahi).

**Networking:** high-bandwidth mehenga link optimize — VRAM (capacity), batching (multiplexing), streaming (progressive delivery), metered cost.

**Interview one-liner:**
> "LLM serving unique: huge model (VRAM/multi-GPU), token-by-token (streaming), expensive GPU (batching), per-token cost. Specialized engines (vLLM/TGI)."

### Section 2 — vLLM Internals (2 innovations)

**Problem base:** KV cache (generated tokens ka state) + GPU idle na rahe.

1. **PagedAttention** — **KV cache** ko OS virtual-memory paging jaisa **fixed pages** me manage. Purana: continuous bada block pehle se reserve (max length) → jawab chhota → waste/fragmentation. Paging: zaroorat pe page allocate → waste kam → zyada concurrency. (≈ packet buffer / MTU-sized chunks.)
2. **Continuous batching** (in-flight) — static batching me ek lamba **request** poore batch ko rokta (baaki slots idle wait). Continuous: slot khaali hote hi naya request add → GPU **never idle** → high throughput. (≈ statistical multiplexing.)

**Result:** 10-24x throughput vs naive.

**⚠️ Term precision:** PagedAttention = **KV cache** manage (not whole VRAM). Continuous batching me lamba **request** (not model) rokta hai.

**Interview one-liner:**
> "vLLM: (1) PagedAttention — KV cache OS-paging jaisa, memory-efficient, more concurrency. (2) Continuous batching — dynamic add/remove requests, GPU never idle. 10-24x throughput. Packet-buffer + statistical-multiplexing jaisa."

**Q&A:** 2 innovations — PagedAttention (KV cache paging, waste kam) + continuous batching (slot khaali → naya request, GPU idle nahi).

### Section 3 — LLM Inference Optimization (7 levers)

Interviewer "latency/cost/throughput kaise optimize" → yeh list:
1. **Quantization** — precision↓ (FP16/INT8/INT4): VRAM + cost + speed down, accuracy thodi kam. (Not fit-only — cost+speed bhi.)
2. **Batching** — requests ek saath → throughput↑ (continuous batching = vLLM).
3. **KV caching** — generated tokens reuse (no full recompute).
4. **Model parallelism/sharding** — bada model multi-GPU (tensor parallel).
5. **Speculative decoding** — chhota fast model kai token guess, bada model ek saath verify → sahi to multi-x speedup, final output bada model ka (**no accuracy loss**).
6. **Prompt caching** — repeated prompt prefix (same system prompt) ka KV reuse.
7. **Distillation** — chhota student model bade teacher se train → fast/sasta serving, lagbhag same quality.

**Trade-off:** zyadatar = speed/cost down, accuracy thodi down (quantization, distillation). Architect ko pata ho kitni accuracy chhod sakte.

**Interview one-liner:**
> "LLM optimization: quantization (cost/VRAM↓), batching (throughput), KV caching, tensor parallelism (big model), speculative decoding (small drafts + big verifies, speedup no accuracy loss), prompt caching (prefix reuse), distillation (small student). Trade-off: speed/cost vs accuracy."

**Q&A:** 3 techniques example — quantization (precision↓, cost+speed+VRAM), speculative decoding (small guesses, big verifies, multi-x, no accuracy loss), distillation (student from teacher, fast/cheap).

### Section 4 — RAG Architecture at Scale

**RAG:** LLM ko relevant docs retrieve karke context me dena (open-book exam) — fresh/private data bina retrain.

**2 phases:**
1. **Ingestion (offline, periodic):** Documents → Chunk → Embed (vector) → Vector DB store. (Index banana.)
2. **Query (online, per request):** Query → Embed → Retrieve top-k (vector DB) → Rerank → assemble context in prompt → LLM → Answer. (Fast lookup + generate.)

**System-design considerations:** vector DB choice (OpenSearch/Pinecone/pgvector), embedding model (self-host vs Titan), chunking (size+overlap — quality), reranking, caching (semantic), latency budget (embed+retrieve+LLM).

**Scale pe:** ingestion async (queue), vector DB sharded/replicated, LLM autoscaled (vLLM), aggressive semantic caching.

**Networking:** multi-stage lookup pipeline — ingestion = route-table populate (index), query = fast lookup + forward (retrieve + generate).

**Interview one-liner:**
> "RAG = LLM + retrieved external docs (fresh/private, no retrain). Offline ingestion (chunk→embed→vector DB) + online query (embed→retrieve→rerank→assemble→LLM). Scale: async ingestion, sharded vector DB, autoscaled LLM, semantic caching. Latency = embed+retrieve+LLM."

**Q&A:** 2 phases — Ingestion (docs→chunk→embed→vector DB store, offline) + Query (query→embed→retrieve top-k→rerank→context→LLM→answer, online).

### Section 5 — Agents at Scale

**Agent:** LLM + tools + reasoning loop (ReAct — LLM sochta → tool call → result → phir sochta... jab tak answer). 

**Scale challenge:** ek agent request = **multiple LLM calls + multiple tool calls** (not 1):
- Latency multiply (5-10x), Cost multiply, **Non-deterministic** (steps unknown — 2 ya 15).

**⭐ #1 RISK = uncontrolled/infinite loop** (agent phasa, steps khatam nahi) → cost+latency blast.

**Controls:**
1. **Max-iteration bound (#1 control)** — max steps (10), phir force stop. = packet **TTL** (har hop count, limit pe drop — loop rokna).
2. **Tool execution** — parallel jahan possible, timeout, error handling.
3. **State management** — conversation/memory in Redis (per-session).
4. **Observability** — per-step tracing (non-deterministic → debug mushkil). = traceroute.
5. **Cost controls** — per-request budget + model tiering (simple=chhota model, complex=bada).

**Networking:** agent loop = multi-hop routing with per-hop decisions. Bound hops = TTL, trace path = traceroute, cost/latency multiply with hops.

**Interview one-liner:**
> "Agents = multiple LLM+tool calls/request → latency/cost multiply, non-deterministic. #1 risk = runaway loop → #1 control max-iterations (= TTL). Plus parallel tools+timeout, Redis state, per-step tracing, cost budget + model tiering. Multi-hop routing jaisa."

**Q&A:** #1 risk = runaway/infinite loop; #1 control = max-iteration bound (= packet TTL — har step count, limit pe stop). Plus budget + model tiering.
