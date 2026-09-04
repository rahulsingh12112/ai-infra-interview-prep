# AISD-07: Auto Scaling + Karpenter (K8s + GPU cost)

> Station 7 — GPU cost ka dil. DevOps angle: HPA, Karpenter, scheduled scaling.

## Paragraph (samajh)
GPU billing = allocated time pe (busy ya idle, bill poora). Peak ke liye 24×7 GPU = paisa jalao. Autoscale: peak pe add, off-peak hatao. K8s (EKS) me: HPA (pods), Karpenter/Cluster Autoscaler (nodes), deployment strategies (canary/blue-green). Karpenter AWS-specific, CA se better GPU scaling (right-sized node direct launch, no pre-defined node groups).

## Points

**1. GPU billing sach**
Allocated (running) time pe, utilization nahi. Launch = poora kiraya (idle bhi). Idle GPU = waste. → autoscale.

**2. K8s 2-level autoscaling**
- **HPA** — pods badhata (same nodes). Trigger: CPU/mem/custom (GPU util, queue depth).
- **Karpenter** — nodes add/remove jab pods ko jagah nahi. Better: right-sized node direct, fast provision, no node-group pre-define.

**3. Scale-to-zero + Spot**
- Scale-to-zero — zero traffic → sab band → zero bill (cold-start trade-off).
- Spot — training 70-90% off, interruption → checkpoint→S3 resume (spot-safe). Inference risky (user-facing).

## Diagram
```
Spike → HPA (pods↑) → Karpenter (right GPU node launch)
Gira  → HPA (pods↓) → Karpenter (khaali node terminate)
Zero traffic → scale-to-zero (zero bill)

Training: spot (70-90% off) + checkpoint→S3
Inference: on-demand (spot risky)
```

## Interview one-liner
> "GPU billing = allocated time (idle bhi bill). HPA (pods, GPU-util/queue metrics) + Karpenter (right-sized GPU nodes, fast). Scale-to-zero off-peak. Spot for training + checkpoint, on-demand inference. Spike = autoscale, not permanent over-provision."

## Practiced Q — din 1000 QPS, raat 50 QPS, cost bacha
A: Baseline (min replicas) = raat 50 QPS handle. HPA + Karpenter autoscale subah peak, phir baseline tak down. Karpenter+HPA (pods+nodes).

⚠️ POLISH (senior signal):
1. **Reactive akela nahi** — GPU node launch = minutes (cold-start). Predictable office hours → **scheduled/predictive pre-scaling** (peak se pehle pre-warm).
2. Baseline size = capacity math (off-peak load). HPA min=baseline, max=peak cap.

⭐ Key line: "Baseline (min replicas)=off-peak. Reactive autoscale (HPA+Karpenter) + scheduled pre-scaling for predictable peaks (cold-start avoid). Max=peak cap."
