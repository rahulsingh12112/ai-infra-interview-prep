# AISD-09: MLOps Loop (Drift → Pipeline → Registry → Deploy)

> Station 9 — Model ko continuously fresh rakhne ka automated cycle.

## Paragraph (samajh)
Monitoring drift pakadta, par pata lagana kaafi nahi — theek bhi karna hai, automatically. MLOps loop: monitor drift pakde → retrain pipeline trigger → train → evaluate (gate) → register → deploy (canary) → wapas monitor. Loop hai, ek-baar ka kaam nahi. 2 critical: evaluation gate (kharab model auto-reject) + order (registry deploy se PEHLE, drift = monitoring output not stage).

## Points

**1. Loop flow (order)**
```
Monitor → Drift detect → Retrain trigger → Train →
Evaluate (GATE) → Register → Deploy (canary) → wapas Monitor
```
Continuous (CI/CD/CT — Continuous Training). Data badalta → model stale → loop fresh rakhta.

**2. Evaluation GATE (critical)**
Train ≠ auto-deploy. Evaluate — baseline se accha? Nahi → auto-reject. Champion (current) vs challenger (naya).

**3. Do common galtiyan**
- Drift ek STAGE nahi — monitoring ka OUTPUT/finding (retrain trigger).
- Registry deploy se PEHLE (train→evaluate→register→deploy). Versioned store (champion/challenger).

## Diagram
```
   ┌──────────────────────────────────────┐
   ▼                                       │
📊 Monitor ──drift──► 🚨 Retrain trigger
                           ▼
                      ⚙️ Train
                           ▼
                      ✅ Evaluate (GATE) ─ NAHI → reject
                           │ HAAN
                           ▼
                      📚 Register (versioned, champ/challenger)
                           ▼
                      🚀 Deploy (canary/blue-green/shadow)
                           └──► wapas Monitor
```

## Deployment strategies (register ke baad)
- Canary — chhote % (5→25→100), gradual
- Blue-green — 2 env, switch, instant rollback
- Shadow — real traffic, output unused, zero-risk validate

## Interview one-liner
> "MLOps loop (CI/CD/CT): monitor→drift→retrain trigger→train→evaluate(GATE)→register→deploy(canary)→monitor. Drift = monitoring output (stage nahi). Registry deploy se pehle. Continuous."

## Practiced Q — stages sahi order + trick
Order: monitor → drift-detect → train → evaluate → register → deploy → monitor ✅
Trick: **drift-detect ek stage NAHI — monitoring ka output** (retrain trigger). Real stages: train/evaluate/register/deploy/monitor.

⭐ Key line: "Drift = monitoring output, standalone stage nahi. Registry deploy se pehle, evaluate register se pehle."
