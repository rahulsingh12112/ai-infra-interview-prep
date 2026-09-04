# AISD-11: Bedrock — Self-host vs Managed

> Station 11 (last) — Architect decision: managed vs self-host. Trade-off articulation.

## Paragraph (samajh)
Ab tak self-hosted stack (apne GPU pe vLLM, khud scale/manage). Doosra raasta = managed (AWS Bedrock) — sirf API call, AWS infra sambhaale. Classic architect decision, koi ek sahi jawab nahi = trade-off. Managed = fast start, low ops, per-call mehnga, kam control. Self-host = sasta at scale, full control, ops burden.

## Points

**1. Bedrock kya (managed GenAI)**
Managed FMs (Claude/Titan) — API, no infra. Sub-services:
- Bedrock base — invoke (on-demand / provisioned throughput)
- Agents — managed agentic
- Knowledge Bases — managed RAG (sirf docs do)
- Guardrails — safety filters

**2. Trade-off**
| | Managed (Bedrock) | Self-host (vLLM/EKS) |
|---|---|---|
| Start | Fast (API) | Slow (setup) |
| Ops | Kam (AWS) | Zyada (tum) |
| Cost | Per-call, mehnga at scale | Sasta beyond breakeven |
| Control | Kam | Full |
| Scaling | AWS auto | Tum |

**3. Decision (breakeven)**
- Low/spiky, fast launch, chhoti team → Managed
- High steady, cost-sensitive, control → Self-host
- Breakeven = volume jahan self-host fixed cost < managed per-call. Zyada → self-host, kam → managed.

## Diagram
```
   Low/spiky ──────────┼────────── High/steady
        ▼           breakeven          ▼
   MANAGED (Bedrock)              SELF-HOST (vLLM/EKS)
   fast, low ops, per-call        cheap@scale, full control, ops burden
```

## Interview one-liner
> "Bedrock = managed FMs (API, no infra) — Agents, Knowledge Bases (managed RAG), Guardrails. Self-host vs managed trade-off: managed (fast, low ops, per-call mehnga, kam control) vs self-host (cheap@scale beyond breakeven, full control, ops burden). Low/spiky→managed, high-steady→self-host."

## Practiced Q
(a) Startup, uncertain traffic, small team, fast launch → **Managed** (infra/scaling/patching provider, pay-per-use). ✅
(b) Enterprise, steady 2000 QPS 24×7, cost-sensitive, custom fine-tuned + control → **Self-host** (beyond breakeven, custom control managed me nahi). ✅

⚠️ POLISH: "obviously" avoid. "I'd lean towards X because..." + trade-off acknowledge (har choice ka cost).

⭐ Key line: "Low/spiky/small/fast → managed. High-steady/cost-sensitive/custom → self-host (beyond breakeven). Trade-off bolo, 'obviously' nahi."
