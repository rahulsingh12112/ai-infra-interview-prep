# SD-00: Overview + Universal Framework

> **Target:** AI Infrastructure Architect (₹1Cr) | **Style:** Paragraph teaching, zero to expert

## Yeh Doc Kyun
System design tere role ka sabse important hissa aur tera sabse bada advantage. Interviewer dekhta tu bade picture me soch sakta ya nahi — components kaise connect, bottleneck kahan, scale kaise, failure me kya, cost kaise bache. Yeh sab tune 15 saal networking/AWS me jiya — bas ML context me apply karna.

## Roadmap
SD-01 Foundations → SD-02 ML System Design → SD-03 LLM/GenAI → SD-04 Production → SD-05 Case Studies.

## Universal Framework (har design me use)
1. Requirements (functional + non-functional, numbers poochho)
2. Scale estimate (QPS, storage, bandwidth)
3. High-level design (boxes + arrows)
4. Deep dive (har component: DB, cache, scaling)
5. Bottlenecks + scale (cache/shard/replicate/queue)
6. Trade-offs (pros/cons)
7. Failure + edge cases (retry, fallback, DR)

**Networking analogy:** Network design jaisa — requirements → capacity → topology → redundancy → failure. Same thinking.
**Interview line:** "Structured: requirements/scale, high-level, deep-dive with trade-offs, bottleneck/failure analysis."

---

## 🎓 Deep Dive — Framework ko Kaise Use Karo (Teacher Session)

> Yeh framework ratne ka nahi, **reflex** banane ka hai. Aaj ke live drill ka nichod.

### Golden Rule: pehle 2-3 min sirf SAWAAL, seedha design NAHI
Interview me koi ek sahi jawab nahi — interviewer dekhta tu **structured** soch raha ya idhar-udhar kood raha. Seedha design shuru = junior signal. **Requirements clarify karna = senior signal.**

### Requirements 3 buckets me poocho (bikhre nahi)
**1. Functional (kya karna hai):**
- Core feature? (chat? multi-turn with history? RAG ya plain LLM? streaming response?)

**2. Scale (numbers — design isi pe khada hota):**
- Users = **total registered ya CONCURRENT** (ek saath online)? — yeh sabse bada, isse QPS nikalta
- Peak QPS? Average request/conversation size?
- Read-heavy ya write-heavy?

**3. Non-functional (quality):**
- **Latency budget** (chat me first-token latency)
- **Availability** target (99.9% = 8.7hr/yr down, 99.99% = 52min/yr)
- **Cost** constraint
- Self-host ya managed (Bedrock/SageMaker)?

### Sabse common miss (yaad rakh)
- **Concurrent ≠ total** — 1M registered ≠ 1M ek saath. "Concurrent kitne + peak QPS" seedha poocho — QPS ka base yehi.
- **Availability/SLA** target poochna.
- **Streaming** chahiye? (token-by-token — design badalta).

### Diagnose vs Design vs Cost — question-type sunna (GPU mock ka gap)
Sawaal ka type pehchano, phir usi lane me:
- **"Design karo"** → framework (requirements → scale → HLD → deep-dive → bottleneck → failure)
- **"Cost zyada, optimize"** → cost levers (batching, quantize, autoscale, spot, cache...)
- **"Slow/down hai, kaise pata karoge"** → DIAGNOSE pehle (check → isolate → profile), phir fix
Sawaal ko galat lane me convert mat karo.

### Networking mapping (tera 15-saal edge)
| SD step | Network design (tune jiya) |
|---|---|
| Requirements | Capacity planning |
| Scale estimate | Bandwidth/traffic sizing |
| High-level design | Topology diagram |
| Deep dive | Per-device config |
| Bottleneck | Congestion/oversubscription analysis |
| Failure/DR | Redundancy (HSRP/VRRP, dual uplink, multi-site) |

### Interview one-liner
> "Main structured chalta: requirements + scale numbers clarify (concurrent users, QPS, latency, availability, cost), phir high-level boxes-arrows, component deep-dive with trade-offs, phir bottleneck + failure analysis. Network design jaisa — requirements → capacity → topology → redundancy → failure."
