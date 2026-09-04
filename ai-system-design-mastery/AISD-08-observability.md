# AISD-08: Observability (3 layers, silent degradation)

> Station 8 — ML monitoring. ⚠️ Rahul ka WEAK SPOT: actuals delayed → prediction distribution.

## Paragraph (samajh)
Normal monitoring simple (CPU/mem/latency/errors). ML twist: model CRASH nahi hota, chup-chaap galat predict karne lagta — system up, dashboard green, predictions kharab = silent degradation. Isliye 3 layers dekhni padti. Plus 3 pillars (metrics/logs/traces).

## Points

**1. ML monitoring 3 layers**
- Infra — GPU util, memory, latency, throughput (Prometheus/Grafana/CloudWatch)
- Model performance — accuracy, prediction distribution (silent degradation catch). Actuals delayed → prediction distribution first defence
- Data quality — input distribution shift?

**2. Observability 3 pillars**
- Metrics — numbers (latency, GPU%)
- Logs — events (errors)
- Traces — request ka poora journey (RAG/agent per-step: embed→retrieve→LLM). OTel/Jaeger/X-Ray.

**3. Silent degradation (ML unique)**
Model up (crash nahi) par galat predict. Infra green, business bura. Uptime kaafi nahi → prediction quality track.

## ⭐ GROUND TRUTH / ACTUALS (Rahul ne 2x miss kiya — CRITICAL)
- Prediction = model ka guess (abhi)
- Ground truth/actuals = asli sach (baad me pata)
- Accuracy = prediction vs actuals. **Actuals bina accuracy IMPOSSIBLE.**
- Fraud me actuals DELAYED — chargeback/investigation hafton baad.
```
Din 0:  Transaction → model "SAFE" (prediction). Sach? PATA NAHI.
Din 30: Chargeback → pata chala FRAUD thi → model galat tha.
Real-time accuracy IMPOSSIBLE (actuals 30 din late).
```
- **Chargeback** = customer "maine nahi kiya" → bank paisa wapas → signal ki fraud thi (hafte lagte).
- Isliye pehla defence = **prediction distribution monitoring** (flag rate shift = turant signal, bina actuals).

## ⚠️ KEY: flag rate gira = BURA
Normally 2% fraud-flag, ab 0.4% → model UNDER-DETECTING → fraud miss kar raha = BAD (achhi cheez NAHI). Flag rate girna = red flag.

## Interview one-liner
> "3 layers: infra, model-perf (accuracy/prediction distribution — silent degradation), data quality. 3 pillars: metrics+logs+traces (OTel). Silent degradation = model up par galat. Actuals delayed (chargeback weeks) → real-time accuracy impossible → prediction distribution first defence."

## Practiced Q — flag rate 2%→0.4%, chargebacks 3 weeks away
(1) Haan problem — flag gira = model under-detecting = fraud miss (bina actuals turant signal)
(2) Accuracy abhi nahi — actuals 3 weeks late
(3) Chargeback aayein → prediction vs actuals compare, precision/recall, drift confirm → retrain

⭐⭐ RATT LO: "Accuracy needs ground truth, often DELAYED → pehla defence PREDICTION DISTRIBUTION, not accuracy. Flag rate gira = under-detecting = BAD."
