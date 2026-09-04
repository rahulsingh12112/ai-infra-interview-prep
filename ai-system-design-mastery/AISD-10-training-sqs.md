# AISD-10: Training Flow + SQS (offline + async)

> Station 10 — Training (offline) vs inference (online), aur async processing.

## Paragraph (samajh)
Ab tak inference/serving dekhi (real-time). Alag duniya = training (offline, background). Training aur inference alag infra: training heavy (GPU, ghante-din, batch, no user wait = throughput), inference fast (real-time, user wait = latency). Training flow: data(S3)→distributed train→checkpoint(S3)→MLflow→registry→deploy. SQS = async kaam (no user wait) — queue me daalo, worker background process kare.

## Points

**1. Training vs Inference — alag infra**
- Training — offline, GPU-heavy, batch, throughput-priority. Distributed + checkpointing.
- Inference — online, real-time, latency-priority. LB + autoscale.
- Bridge: registry + deploy pipeline.

**2. Training flow**
```
Data(S3) → Distributed train (parallelism, NVLink+IB)
→ Checkpoint→S3 (spot-safe) → MLflow track → Registry → deploy
```
- Checkpoint — fail/spot-interrupt pe shuru se nahi, resume. S3.
- Spot — 70-90% sasta, interruption checkpoint se. Training ideal.

**3. SQS — async**
Producer → queue (buffer) → consumer (own speed). 3 fayde: decouple, load smoothing (spike absorb), reliability (worker down = safe). ML: doc ingestion, batch inference, training submit. Autoscale workers on queue depth.

## Diagram
```
TRAINING (offline):
  Data(S3) → Distributed train → Checkpoint→S3 (resume) → MLflow → Registry → deploy

SQS (async):
  Producer → [📥 Queue] → Consumer (worker)
             spike absorb   decouple + reliable
```

## Interview one-liner
> "Training = offline, GPU-heavy, throughput (distributed + checkpoint→S3 spot-resume). Inference = online, latency. SQS async: producer→queue→consumer (decouple + load smoothing + reliability). No user wait → async."

## Practiced Q
(a) 50k docs/day chunk+embed → **async/SQS** (no user wait, queue+workers). +load smoothing, autoscale workers on queue depth. ✅ 10/10
(b) 12hr spot training, 8th hr interrupt → poora kaam NAHI gaya. Checkpoint→S3 har ghante → ~1hr rework, 7th-hr checkpoint se resume. ✅ 10/10

⭐ Key line: "No user wait → async (SQS). Spot = sasta + always checkpoint→S3 (resume not restart). Checkpoint freq = overhead vs rework trade-off."
