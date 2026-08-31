# LS-05: Optimization, Benchmarking & Interview Mastery (Zero to Expert)

> Sab tie karta — cost/latency optimization, benchmarking, aur interview me poori tarah ready hone ke liye consolidated Q&A + ek full serving design walkthrough.

---

## 1. Latency Optimization (Consolidated)

LLM serving me latency kam karne ke saare levers ek jagah: **Streaming** (SSE — token-by-token user ko, perceived latency bahut kam, user turant text dekhta bhale total time same). **Batching** (continuous batching — throughput, par batch size latency trade-off, tune karo). **KV cache + prompt caching** (recompute avoid). **Quantization** (chhota model = faster). **Speculative decoding** (draft model speedup). **Model choice** (chhota/distilled model faster for simple tasks). **Region proximity** (user ke paas deploy — network latency kam). **Warm pools** (pre-loaded models, no cold-start). TTFT (time-to-first-token) optimize karna perceived latency ke liye sabse important.

**Interview line:** "Latency levers — streaming (perceived, TTFT focus), continuous batching (tune size), KV/prompt caching, quantization, speculative decoding, smaller model for simple, region proximity, warm pools. TTFT most impactful for UX."

---

## 2. Cost Optimization (Consolidated — tera FinOps)

LLM serving cost ke levers: **Batching** (GPU utilization max — same GPU zyada requests = cost/request down). **Quantization** (sasta/chhota GPU). **Model tiering** (simple queries cheap model, complex expensive — routing). **Caching** (repeated queries no compute — big saver). **Spot for batch/training** (70-90% off). **Autoscaling + scale-to-zero** (idle pe cost nahi). **Self-host vs managed breakeven** (high volume self-host cheaper). **Right-sizing GPU** (over-provisioned mat). **Provisioned throughput** (Bedrock — steady high volume pe on-demand se sasta). Monitor via cost dashboards + per-model/team tags.

**Interview line:** "Cost — batching (GPU utilization), quantization (cheaper GPU), model tiering, caching (big saver), spot for batch, autoscale + scale-to-zero, self-host breakeven, right-size, provisioned throughput for steady. Monitor + tag attribution."

---

## 3. Benchmarking & Capacity Planning

Serving deploy karne se pehle benchmark: **load testing** (locust/k6 se realistic traffic simulate), measure p50/p99 latency + throughput + GPU utilization at different loads, find **saturation point** (kis QPS pe latency degrade). Isse **capacity plan** — required QPS ÷ per-instance capacity = instances needed (+ buffer for spikes + HA). LLM me tokens/sec measure karo (input + output length matter). Yeh architect ka data-driven decision — guess nahi, benchmark.

**Networking analogy:** Yeh capacity planning jaisa jo tu karta — load test, find saturation, size for peak + redundancy. Bandwidth planning ka ML version.

**Interview line:** "Benchmark before deploy — load test (locust/k6), measure p99 latency + throughput + GPU-util at loads, find saturation. Capacity = required-QPS ÷ per-instance-capacity + buffer + HA. Data-driven, not guess."

---

## 4. FULL DESIGN WALKTHROUGH — "Serve Llama-3-70B to Production"

Interview me poora question aaye to aise attack kar (framework se):

**Requirements:** Llama-3-70B (huge — ~140GB FP16), enterprise chatbot, 500 QPS peak, spiky (business hours), p99 < 3s acceptable, streaming, cost-conscious, data must stay in-account (compliance).

**Key decision — managed vs self-host:** Data compliance (in-account) + 70B (Bedrock me available but cost) + high volume → lean **self-host on EKS** (control + data + cost at scale). (Agar compliance na hoti aur volume low, Bedrock simpler.)

**Model fitting:** 70B FP16 = 140GB, ek GPU (A100 80GB) me nahi → **tensor parallelism** across 2-4 GPUs, ya **quantize to INT8** (~70GB, fewer GPUs). Decision: INT8 quantization (minor accuracy loss acceptable for chat) + tensor parallelism = 2x A100.

**Architecture:** API Gateway (auth/rate-limit) → ALB → vLLM pods on EKS (GPU node group, tensor-parallel-size=2, INT8) → streaming SSE response. Redis prompt cache. Prometheus/Grafana monitoring. Karpenter autoscaling (spiky → scale GPU nodes on demand, scale-in off-hours).

**Scale/bottleneck:** GPU is bottleneck. vLLM continuous batching maximizes per-GPU throughput. HPA on GPU-util + queue depth. Karpenter adds GPU nodes on spike (pre-warm buffer for provisioning delay). Cache reduces load (repeated queries).

**Cost:** Spiky → autoscale + scale-in off-hours (don't run peak capacity 24/7). INT8 (fewer GPUs). Caching. Spot for non-critical/batch. Monitor per-team.

**Failure/HA:** Multi-AZ GPU node groups. Pod fail → reschedule. Node fail → Karpenter replaces. Overload → queue + rate-limit + graceful (smaller model/cached fallback). Model load fail → pod not-ready.

**Trade-offs stated:** Self-host (cost/control/compliance) vs Bedrock (ops) — chose self-host for compliance + volume. INT8 (cost) vs FP16 (accuracy) — chose INT8, minor loss acceptable for chat. Batching (throughput) vs latency — tuned.

**Interview delivery:** "Self-host on EKS for data compliance + cost at volume. 70B → INT8 quantization + tensor-parallel across 2 A100s. API Gateway → ALB → vLLM pods (continuous batching, streaming) → Redis cache. Karpenter autoscale for spiky (scale-in off-hours). GPU bottleneck — batching + autoscale + cache. Multi-AZ HA, graceful fallback. Trade-offs: self-host vs Bedrock (compliance won), INT8 vs FP16 (cost won, minor accuracy)."

---

## 5. Complete Interview Q&A (LLM Serving — all files)

**Q: Serve a large LLM at scale — approach?** — (Full walkthrough above — framework: requirements → managed/self-host → model fitting → architecture → scale → cost → failure → trade-offs.)

**Q: vLLM ke innovations?** — "PagedAttention (memory-efficient KV cache) + continuous batching (GPU never idle). 10-24x throughput."

**Q: Managed vs self-host LLM?** — "Bedrock: zero ops/fast, per-token cost, less control. Self-host (vLLM/EKS): cheaper at scale/full control/data-in-account, ops burden. Volume + compliance + team decides. Hybrid common."

**Q: 70B model ek GPU me nahi — kya?** — "Tensor parallelism (multi-GPU, NVLink) + INT8 quantization (footprint down). vLLM tensor_parallel_size."

**Q: LLM cost optimize?** — "Batching (GPU-util), quantization, model tiering, caching (big saver), autoscale+scale-to-zero, spot, self-host breakeven, provisioned throughput. Monitor + tags."

**Q: LLM latency optimize?** — "Streaming (perceived/TTFT), batching (tuned), caching, quantization, speculative decoding, smaller model, region proximity, warm pools."

**Q: SageMaker inference options?** — "Real-Time (steady low-latency), Serverless (spiky, scale-to-zero, cold-start), Async (big payload), Batch (bulk offline)."

**Q: K8s GPU serving?** — "NVIDIA plugin (nvidia.com/gpu), GPU node pools + taints, HPA (GPU-util/queue) + Cluster Autoscaler/Karpenter, KServe/vLLM, EKS + IRSA + ALB."

**Q: Benchmark serving?** — "Load test (locust/k6), measure p99+throughput+GPU-util, find saturation, capacity = QPS ÷ per-instance + buffer + HA."

---

## Tera LLM Serving DONE!

6 files — fundamentals se production AWS/K8s serving tak, optimization, aur full design walkthrough. System Design doc ke saath yeh tera "model serving" gap poori tarah bhar deta.

**Revision priority:** LS-02 (vLLM/optimization — technical depth), LS-03 (AWS — tera stack), LS-04 (K8s — tera strength), LS-05 (design walkthrough — interview). LS-01 foundation.

**Interview power move:** LS-05 ka full walkthrough ratta+samajh — koi bhi "serve X model" question isi framework se attack kar. Aur jahan K8s/networking/cost/HA aaye — tera 15-saal edge dikha.
