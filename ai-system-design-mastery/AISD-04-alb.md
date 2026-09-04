# AISD-04: Application Load Balancer (ALB)

> Station 4 — Horizontal scale ka traffic distributor. LLM me least-connections key.

## Paragraph (samajh)
Horizontal scale = ek server ki jagah kai servers/GPU nodes. Sawaal: request kaunse pe jaaye? LB baant deta — ek pe load na aaye, baaki khaali na baithe. LLM twist: normal request predictable (ms), LLM request time vary karta (2 line=100ms, 2000 tokens=10s). Isliye round-robin theek nahi — kuch servers lambi requests me jam. ALB least-connections use karta. Plus health checks — dead server detect → traffic band (auto-failover).

## Points

**1. LB core — distribute + health check**
Kai servers ke aage. Healthy server pe bhejta. Health check → dead detect → traffic band (failover). HA ka detection+failover pillar.

**2. Algorithms — kaunsa kab:**
- Round-robin — baari-baari. Same-duration ke liye. LLM me lambi requests jam.
- Least-connections — sabse kam active requests wale pe. Uneven duration ka solution. LLM best.
- Weighted — capacity ke hisaab se. Mixed hardware ke liye.

**3. LLM me least-connections kyun (GOLD)**
LLM uneven duration (token count vary). Round-robin blindly baari-baari → ek server 3 lambi requests me jam, dusra free. Least-connections load-aware → free dhoondhta.

## Diagram
```
              ⚖️ ALB (least-connections + health checks)
              │  "kis server pe kam active load?"
     ┌────────┼────────┐
     ▼        ▼        ▼
  Node A1   Node A2   Node B1
  (2 act)   (5 act)   (1 act) ◄── nayi request (sabse kam)
     │
  health check fail? → traffic band (failover)
```

## Interview one-liner
> "LB distribute + health checks (dead detect → failover). Round-robin (equal-duration), least-connections (uneven — LLM best, load-aware), weighted (mixed capacity). LLM uneven (token count) → round-robin jam → least-connections."

## Practiced Q — 5 identical A100 servers, mixed jawab length
A: Least-connections. Kyun: uneven duration, round-robin me lambi-request node pe naya request wait karega jabki dusra idle; least-conn = GPU hamesha busy (utilization max). Weighted NAHI — hardware same, capacity-difference ka use-case absent.

## Bonus (advanced)
vLLM + continuous batching: routing queue-depth / pending-tokens / KV-cache-utilization pe bhi ho sakti (least-outstanding-requests). Default answer = least-connections; push kare to "vLLM me queue-depth/KV-cache-aware routing."

⭐ Key line: "Uneven duration → least-connections (load-aware). Weighted sirf capacity alag ho. Advanced: KV-cache-aware routing."
