# AISD-03: Caching (CloudFront + ElastiCache)

> Station 3 — Do-level caching. LLM me BIGGEST cost + latency lever.

## Paragraph (samajh)
Caching = jo cheez baar-baar chahiye aur mehngi hai, compute karke reuse karo, dobara mat banao. LLM me bada lever kyunki har call = GPU compute = paisa + latency. Ek hi sawaal 1000 log → 1000 GPU calls = waste. Do jagah: CloudFront (CDN, edge, user ke paas — static + cached) aur ElastiCache/Redis (app ke paas — LLM prompt/response). LLM special trick: semantic cache (same-meaning queries bhi hit).

## Points

**1. CloudFront (CDN) — edge, user ke paas**
Static content (JS/CSS/images) + cached responses user ke nazdeek edge se. Origin tak request jaati hi nahi. Latency + origin load down.
⚠️ Exact URL/KEY pe caching. Semantic matching NAHI kar sakta (embedding nahi banata).

**2. ElastiCache (Redis) — app-level**
In-memory. LLM prompt→response store. Same prompt → LLM skip. Biggest LLM cost saver.

**3. Exact-match vs Semantic (LLM-specific)**
- Exact-match: bilkul same text → hit. Rigid ("return policy?" ≠ "wapsi policy?").
- Semantic: query → embedding → similar-meaning cached prompts se match. Varied phrasing, same matlab → hit. Common-question workload ke liye best.

## ⚠️ KEY CORRECTION (Rahul ki galti yahan)
CDN dynamic LLM queries (varied phrasing) cache NAHI kar sakta — har text alag = har query "naya" = no hit. Sahi split:
```
Static (UI, images, fixed FAQ pages)  → CloudFront (exact key, edge)
Dynamic queries, varied phrasing       → Semantic cache (Redis + embeddings)  ← HERO
Dynamic queries, identical text        → exact-match (Redis) — varied phrasing me kaam nahi
```

## Diagram (two-level)
```
User
  ▼
🌐 CloudFront (edge)  ── static + cached ──► HIT? turant
  │  MISS
  ▼
App / ALB
  ▼
⚡ Redis (semantic/exact)  ── prompt/response ──► HIT? LLM skip
  │  MISS
  ▼
🖥️ GPU (LLM)  ── compute → response ── (store in Redis)
```

## Interview one-liner
> "Two-level: CloudFront (edge, static + exact-key cached — latency+origin load down) + Redis (app prompt/response cache — LLM skip, biggest cost saver). LLM: exact-match (rigid) vs semantic (embedding similarity, same-meaning hit). Repeated-query workload = caching biggest cost+latency lever."

## Practiced Q — chatbot, 70% wahi 20 Qs, alag phrasing
A: CloudFront = sirf static (UI/images). Dynamic varied-phrasing queries = **semantic cache (Redis+embeddings)** = primary hero. ~70% hit → ~70% GPU cost saving + latency ms vs sec.

⭐ Key line: "CDN = exact-key static caching (semantic nahi). Varied-phrasing same-meaning = semantic cache (Redis+embeddings) primary. CloudFront sirf static."
