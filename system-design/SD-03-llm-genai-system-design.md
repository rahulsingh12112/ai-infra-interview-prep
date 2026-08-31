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
