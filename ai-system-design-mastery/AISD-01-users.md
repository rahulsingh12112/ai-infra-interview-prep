# AISD-01: Users (concurrent vs total, QPS base)

> Station 1 — Request flow ka pehla box. Poore design ki neenv.

## Paragraph (samajh)
Kisi bhi system design me pehla box users hota hai, par asli cheez number hai. "10 lakh users" bolna trap hai — kyunki 10 lakh **registered** ka matlab 10 lakh **ek saath online** nahi. Design hamesha **concurrent load** pe khada hota hai. Concurrent se **QPS** nikalta hai, aur QPS se poora sizing — kitne servers, GPU, cache. Isliye ye chhota box poore design ki neenv hai. Senior yahin ruk ke numbers clarify karta, junior seedha design banane lagta.

## Points

**1. Concurrent ≠ Total**
Total = registered (10 lakh). Concurrent = ek waqt me actually online/active (maybe 10-50 hazaar). Design concurrent pe hota. Interview me sabse pehla clarify karo: "concurrent users kitne?"

**2. QPS nikalna**
```
QPS = concurrent users × (requests per user per second)
```
Example: 10,000 concurrent, avg 30 sec me 1 request → QPS = 10,000 × (1/30) ≈ 333 QPS average.

**3. Peak vs Average**
Average pe design karoge to peak pe system girega. Hamesha **peak QPS** pe size karo. Autoscaling peak-average gap handle karta.

## Peak factor — DEPTH (fixed nahi hota!)
Peak factor universal constant nahi — traffic pattern pe depend:
- **Steady** (internal tool, B2B, logs) → 1.5-2x
- **Consumer, time-zone concentrated** (India shaam 7-10) → 3-5x
- **Event-driven / viral** (IPL, flash sale, Diwali, breaking news) → **10x-50x+**

⚠️ Interview me factor **maano mat — poocho ya justify karo**. Galat maan ke design → peak pe crash.

Handle:
- Predictable bada peak → capacity provision (baseline zyada)
- Unpredictable spike → autoscale + queue (SQS) + rate limit + graceful degradation (24×7 over-provision mehenga — GPU billing = allocated time)

## Diagram (kaise sochna)
```
10,00,000 registered   (sizing me MAT use karo)
        │  (kitne ek saath active?)
        ▼
   10,000 concurrent
        │  × (1 req / 30 sec)
        ▼
   ~333 QPS average
        │  × 2-3 (peak factor — pattern pe depend!)
        ▼
   ~1000 QPS peak   ← design isi pe khada
```

## Interview one-liner
> "Pehle concurrent users clarify karunga, total nahi — design concurrent pe. QPS = concurrent × req-rate. Phir peak factor (pattern pe depend, assume nahi karta — poochta) lagाke peak QPS — sizing usi pe."

## Practiced Q
Q: 5 lakh registered, peak 20k concurrent, 1 req/20 sec → Peak QPS?
A: Registered ignore. 20,000 × (1/20) = **1000 QPS**. (Concurrent already peak, factor nahi lagाya.)

## Interview polish
Answer sirf number mat do — **process bolo**: "Registered ignore, concurrent × req-rate = 1000 QPS, design isi pe." Process = senior signal.
