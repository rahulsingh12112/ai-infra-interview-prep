# LS-02: LLM Serving Deep Dive (Zero to Expert)

> Yeh core. LLM serving ke unique challenges aur solutions — vLLM, batching, KV cache, quantization, parallelism. Interview me yeh depth ₹1Cr dilati.

---

## 1. LLM Serving Kyun Special — The Core Problem

Classic ML model chhota (MBs), ek forward pass, ek output. LLM bilkul alag: model **bahut bada** (7B params ~14GB in FP16, 70B ~140GB — multiple GPUs), aur output **token-by-token autoregressive** — matlab ek word generate karta, phir usko wapas input me daal ke agla, phir agla... 500-token response = 500 sequential forward passes. Yeh **slow aur compute-heavy** hai. Plus har generation me pichle tokens ka **KV cache** (key-value attention state) memory me rakhna padta — yeh GPU memory kha jaata. In teen problems (size, sequential generation, memory) ko solve karna LLM serving ka poora khel hai.

**Networking analogy:** Classic ML = single packet forward (fast). LLM = ek lambi session jahan har packet pichle pe depend karta (stateful, sequential) + state (KV cache) memory me maintain — jaise stateful firewall connection tracking, par bahut heavy.

**Interview line:** "LLM special — huge model (multi-GPU), autoregressive token-by-token (sequential, slow), KV cache (memory-heavy). In teen problems (size/sequential/memory) ko solve karna serving ka core."

---

## 2. KV Cache — Samajhna Zaroori

Jab LLM token generate karta, har token ke liye attention computation hoti jisme pichle sab tokens ke "key" aur "value" vectors chahiye. Agar har naye token pe yeh dobara compute karo to bahut waste (same pichle tokens baar-baar). Isliye **KV cache** — pichle tokens ke K/V vectors memory me store karke reuse. Isse speed badhti par **GPU memory** consume hoti (lambe conversations = bada cache). Yeh memory management LLM serving ka central challenge — kitni requests ek GPU pe fit karengi yeh KV cache management pe depend karta.

**Interview line:** "KV cache — generated tokens ke attention key/value vectors store karke reuse (har token pe recompute avoid). Speed up, par GPU memory heavy. Memory management = kitni concurrency fit hogi."

---

## 3. vLLM — 2 Key Innovations (INTERVIEW GOLD)

vLLM sabse popular open-source LLM serving engine. Do innovations jo isko 10-24x faster banate:

**PagedAttention:** Traditional serving me KV cache ko ek contiguous memory block me rakhte the — problem yeh ki har request ko max-possible memory pre-allocate karni padti (waste), aur fragmentation hoti. vLLM ne **operating system ke virtual memory paging** se idea liya — KV cache ko chhote fixed-size "pages/blocks" me toda, jo non-contiguously allocate ho sakein aur jitni chahiye utni. Memory waste ~4% tak aa gaya (pehle 60-80% waste hota tha), matlab **bahut zyada requests ek GPU pe fit**.

**Continuous Batching (in-flight batching):** Traditional "static batching" me batch ke sab requests ke complete hone ka wait hota — ek slow request (lamba response) poore batch ko rok deta, GPU idle. vLLM **continuous batching** karta — jaise-jaise koi request complete hoti, uski jagah nayi request turant batch me add ho jaati. GPU kabhi idle nahi, throughput maximize.

In dono se vLLM naive serving se **10-24x throughput** deta same hardware pe.

**Networking analogy:** PagedAttention = memory ko fixed pages me manage (packet buffer/MTU management — fragmentation avoid, efficient allocation). Continuous batching = statistical multiplexing — link kabhi idle nahi, slot khaali hote hi naya traffic bharo (jaise TDMA vs dynamic multiplexing).

**Interview line:** "vLLM — PagedAttention (KV cache ko OS-paging jaisa fixed blocks me, memory waste 60% se 4%, zyada concurrency) + continuous batching (request complete hote hi nayi add, GPU never idle). 10-24x throughput vs naive."

---

## 4. Quantization (Cost/Memory ka bada lever)

Model ke weights normally FP32/FP16 (32/16-bit floats) me hote. **Quantization** matlab inhe lower precision me convert karo — **INT8** (8-bit) ya **INT4** (4-bit). Isse model ka size aur memory footprint dramatically kam (FP16 se INT4 = 4x chhota), inference faster, sasta hardware pe chal jaata. Trade-off: thodi **accuracy loss** (usually minimal for INT8, thodi zyada INT4). Techniques: GPTQ, AWQ (popular for LLMs), bitsandbytes. Architect decision: kitni accuracy sacrifice acceptable vs cost/speed gain. Production me INT8 often sweet spot.

**Interview line:** "Quantization — weights FP16 → INT8/INT4, model 2-4x chhota, faster, sasta GPU. Trade-off: minor accuracy loss (INT8 minimal). GPTQ/AWQ. INT8 often sweet spot cost-vs-accuracy."

---

## 5. Model Parallelism (Bade Models Multi-GPU)

Jab model ek GPU ki VRAM me fit nahi (70B+), usko multiple GPUs pe split karna padta. Types: **Tensor parallelism** (ek layer ke computation ko GPUs me split — intra-layer, high communication, same node me NVLink se). **Pipeline parallelism** (model ke alag layers alag GPUs pe — stage 1 GPU-A, stage 2 GPU-B). **Data parallelism** (serving me kam relevant — training me zyada). Serving me aksar tensor parallelism within a node (NVLink fast) use hota bade models ke liye. vLLM tensor parallelism support karta (`tensor_parallel_size`).

**Networking analogy:** Tensor parallelism = ek computation ko parallel links pe split (high inter-link communication — NVLink = backplane). Pipeline = assembly line stages across nodes.

**Interview line:** "Bada model (70B+) ek GPU me nahi — tensor parallelism (layer split intra-node, NVLink) ya pipeline (layers across GPUs). vLLM tensor_parallel_size. Inter-GPU interconnect (NVLink/InfiniBand) bottleneck."

---

## 6. Other Optimizations

**Speculative decoding:** ek chhota fast "draft" model kai tokens guess karta, bada model unhe ek saath verify (accept/reject) — sequential bottleneck kam, speedup. **Prompt caching:** common prompt prefixes (system prompt) ka KV cache reuse across requests. **FlashAttention:** attention computation ka memory-efficient + fast implementation. **Chunked prefill:** lambe prompts ko chunks me process. Yeh sab latency/throughput optimize karte — architect ko awareness chahiye.

**Interview line:** "Extra opts — speculative decoding (draft model guesses, big verifies — speedup), prompt caching (shared prefix KV reuse), FlashAttention (efficient attention), chunked prefill. Latency/throughput levers."

---

## 7. vLLM Hands-On (Practical)

```bash
pip install vllm

# Serve a model (OpenAI-compatible API)
python -m vllm.entrypoints.openai.api_server \
  --model meta-llama/Llama-3-8B-Instruct \
  --tensor-parallel-size 1 \        # 1 GPU (bada model = zyada)
  --max-model-len 8192 \
  --gpu-memory-utilization 0.9      # 90% VRAM use
```
```python
# Client (OpenAI-compatible)
from openai import OpenAI
client = OpenAI(base_url="http://localhost:8000/v1", api_key="dummy")
resp = client.chat.completions.create(
    model="meta-llama/Llama-3-8B-Instruct",
    messages=[{"role": "user", "content": "Hello"}],
    stream=True,   # streaming
)
```

> **Note (tera Mac):** vLLM ko NVIDIA GPU chahiye — tera Mac pe local nahi chalega. Yeh cloud (EC2 GPU/SageMaker/EKS GPU node) pe chalega. Concept samajhne ke liye theory kaafi; hands-on cloud GPU pe.

**Interview line:** "vLLM serve — OpenAI-compatible API, tensor-parallel-size (multi-GPU), gpu-memory-utilization, max-model-len. Client OpenAI SDK se, streaming. Needs NVIDIA GPU — cloud (EC2/EKS GPU)."

---

## Interview Q&A (LLM Serving Deep)

**Q: LLM serving special kyun?** — "Huge model (multi-GPU), autoregressive token-by-token (sequential/slow), KV cache (memory-heavy). In teen ko solve karna."

**Q: vLLM fast kyun?** — "PagedAttention (KV cache OS-paging, waste 60%→4%, concurrency up) + continuous batching (GPU never idle). 10-24x."

**Q: Quantization?** — "Weights FP16→INT8/INT4, 2-4x chhota+faster+sasta, minor accuracy loss. GPTQ/AWQ. INT8 sweet spot."

**Q: Bada model ek GPU me nahi to?** — "Tensor parallelism (layer split, NVLink) ya pipeline. vLLM tensor_parallel_size. Interconnect bottleneck."

**Q: KV cache kya?** — "Generated tokens ke K/V vectors store+reuse, recompute avoid. Speed up, memory heavy — concurrency limit."
