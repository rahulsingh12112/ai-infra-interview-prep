# LS-01: Model Serving Fundamentals (Zero to Expert)

> Serving ke universal concepts — LLM ho ya classic ML. Foundation pakka to LLM-specific (LS-02) easy.

---

## 1. Serving Kya Hai — Aur Kyun Alag Challenge

Model train karna ek baar ka kaam hai (offline, batch, GPU pe ghante). Serving matlab us trained model ko **live** rakhna taaki users real-time predictions le sakein. Yeh alag challenge hai kyunki ab **latency matter karti** (user wait nahi karega), **scale matter karta** (hazaaron concurrent requests), **availability matter karti** (24/7 up), aur **cost matter karta** (har prediction compute, especially GPU pe mehenga). Ek architect ko in chaaron ko balance karna padta. Training me tu socht "kitna accurate", serving me "kitna fast, kitna sasta, kitna scalable, kitna reliable."

**Networking analogy:** Training = network design/planning phase (offline, heavy). Serving = live traffic forwarding (real-time, must be fast + always up + efficient). Tu jaanta live network chalana planning se alag khel hai.

**Interview line:** "Serving = trained model live for real-time predictions — latency, scale, availability, cost balance. Training offline/heavy, serving online/continuous. Alag concerns."

---

## 2. Serving Patterns (4 types — kab kya)

**Online (real-time):** request aaya, turant response. Latency-critical (chatbot, fraud). Model loaded on server, REST/gRPC endpoint, always-on, LB ke peeche. Sabse common, sabse challenging.

**Batch:** bahut predictions ek saath, offline (raat ko sab users ke recommendations). Latency irrelevant, throughput important. Scheduled job, results stored.

**Streaming:** continuous data stream pe predict (Kafka se aata data). Event-driven.

**Serverless:** spiky/unpredictable traffic. Idle pe cost nahi (scale-to-zero), spike pe auto-scale. Trade-off: cold-start latency (pehli request slow). SageMaker Serverless Inference.

Decide karne ke liye: latency requirement, traffic pattern, cost. Steady high-traffic → always-on servers. Spiky → serverless. Bulk → batch.

**Interview line:** "Online (real-time, always-on, LB), batch (bulk offline, throughput), streaming (continuous), serverless (spiky, scale-to-zero, cold-start trade-off). Choose by latency + traffic pattern + cost."

---

## 3. Serving Metrics (yeh bolna aata ho)

Serving performance measure karne ke metrics: **Latency** — p50/p95/p99 (percentiles — p99 = worst 1% experience, yeh matter karta, average nahi). **Throughput** — requests/sec (RPS) ya LLM me tokens/sec. **TTFT** (Time To First Token — LLM streaming me pehla token kitni jaldi, perceived latency). **TPOT** (Time Per Output Token — har subsequent token). **Concurrency** — kitne parallel requests. **GPU utilization** — cost efficiency (idle = waste). Interview me p99 latency aur throughput dono discuss karna maturity dikhata (average latency deceptive hoti).

**Networking analogy:** p99 latency = worst-case RTT (jitter matter karta, average nahi). Throughput = bandwidth. GPU utilization = link utilization (maximize).

**Interview line:** "Metrics — latency (p50/p95/p99, not average), throughput (RPS/tokens-sec), LLM: TTFT + TPOT, concurrency, GPU utilization. p99 matters most for user experience."

---

## 4. Model Formats & Serialization

Model ko serve karne ke pehle standard format me hona chahiye. Common: **Pickle/joblib** (Python, sklearn — simple par Python-locked, security risk), **ONNX** (Open Neural Network Exchange — framework-agnostic, optimized inference, portable), **TorchScript/SavedModel** (PyTorch/TF native), **MLflow model** (packaged with env + signature — tera MLflow doc). Production me ONNX popular kyunki framework-independent + runtime-optimized. LLM ke liye special formats (safetensors — safe, fast loading; GGUF for quantized).

**Interview line:** "Formats — pickle (simple, Python-locked), ONNX (framework-agnostic, optimized, portable), native (TorchScript/SavedModel), MLflow (packaged). LLM: safetensors, GGUF (quantized). ONNX for portable optimized inference."

---

## 5. Serving Frameworks/Servers

Model serve karne ke tools: **TorchServe** (PyTorch), **TensorFlow Serving** (TF), **Triton Inference Server** (NVIDIA — multi-framework, GPU-optimized, dynamic batching, popular), **KServe** (Kubernetes-native serving, autoscaling), **BentoML** (Python, easy packaging), aur LLM-specific: **vLLM, TGI (Text Generation Inference), TensorRT-LLM** (LS-02). Classic ML ke liye Triton/TorchServe/FastAPI; LLM ke liye vLLM/TGI. Architect ko pata hona kaunsa kab.

**Interview line:** "Servers — Triton (multi-framework, GPU, dynamic batching), TorchServe/TF-Serving (native), KServe (K8s-native + autoscale), BentoML (packaging). LLM: vLLM/TGI/TensorRT-LLM. Choose by framework + scale + K8s."

---

## 6. Simple Serving — FastAPI (Baseline)

Sabse basic serving — model ko REST API me wrap:
```python
from fastapi import FastAPI
import mlflow.pyfunc

app = FastAPI()
model = mlflow.pyfunc.load_model("models:/my-model@champion")  # startup pe load

@app.post("/predict")
def predict(data: dict):
    prediction = model.predict(data["inputs"])
    return {"prediction": prediction.tolist()}
```
Yeh chhote scale/classic ML ke liye theek — FastAPI + Docker + K8s. Par LLM ke liye insufficient (no batching/optimization) — vLLM chahiye (LS-02).

**Interview line:** "Basic serving — model FastAPI me wrap, load at startup, /predict endpoint, Docker + K8s. Classic ML ke liye theek; LLM ke liye vLLM (batching/optimization) chahiye."

---

## Interview Q&A (Fundamentals)

**Q: Serving patterns?** — "Online (real-time, always-on), batch (bulk offline), streaming (continuous), serverless (spiky, scale-to-zero, cold-start). By latency+traffic+cost."

**Q: Kaunse metrics?** — "p99 latency (not average), throughput (RPS/tokens-sec), LLM: TTFT+TPOT, GPU utilization. p99 = user worst-case."

**Q: ONNX kyun?** — "Framework-agnostic, runtime-optimized, portable. Train PyTorch, serve ONNX optimized anywhere."

**Q: Serving framework choice?** — "Triton (multi-framework GPU), KServe (K8s), LLM: vLLM/TGI. By framework, scale, platform."
