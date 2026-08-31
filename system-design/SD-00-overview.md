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
