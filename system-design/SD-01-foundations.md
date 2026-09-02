# SD-01: System Design Foundations (Zero to Expert)

> Universal principles — yeh kisi bhi system pe lagte, ML ho ya nahi. Foundation pakka hoga to ML/LLM design easy.

---

## 1. Scalability — Vertical vs Horizontal

Scalability ka matlab hai system load badhne pe handle kar paaye. Do tarike hote hain. **Vertical scaling** (scale up) matlab ek hi machine ko aur powerful bana do — zyada CPU, RAM, GPU. Yeh simple hai par ek limit ke baad ruk jaata (ek machine kitni bhi badi ho, physical limit hai) aur mehenga hota, plus single point of failure. **Horizontal scaling** (scale out) matlab aur machines add kar do aur load unme baant do. Yeh practically unlimited scale deta aur fault-tolerant hai (ek machine mare to baaki chalte rahein), par isme complexity aati — load balancing, data consistency, coordination sambhaalna padta.

Modern systems (aur tera ML infra) hamesha **horizontal scaling** prefer karte kyunki scale aur resilience dono chahiye. Ek LLM serving platform 1000 requests/sec handle kare, to tu ek giant server nahi lagayega — tu kai inference servers lagayega ek load balancer ke peeche.

**Networking analogy:** Vertical = ek router ko bigger chassis se replace karna. Horizontal = kai routers/links add karke load-share karna (ECMP jaisa). Tu jaanta scale pe horizontal hi chalta.

**Interview line:** "Horizontal scaling prefer karta for scale + fault tolerance; vertical ki physical limit aur SPOF hoti. Trade-off: horizontal me consistency/coordination complexity."

---

## 2. Latency vs Throughput

Yeh do alag cheezein hain jo log confuse karte. **Latency** = ek single request kitni jaldi complete hoti (e.g., 200ms). **Throughput** = system kitne requests per second handle kar sakta (e.g., 1000 QPS). Dono me aksar trade-off hota. Jaise LLM serving me, agar tu requests ko **batch** karke ek saath GPU pe bhejta, to throughput badh jaata (GPU efficiently use hota) par individual request ki latency thodi badh jaati (usko batch bharne ka wait karna padta). Architect ko decide karna padta application ke hisaab se — real-time chat me latency important, bulk document processing me throughput.

**Networking analogy:** Latency = ek packet ka round-trip time (ping). Throughput = link ki bandwidth (Gbps). Tu jaanta — low latency aur high throughput dono optimize karna alag challenges hain.

**Interview line:** "Latency = single request speed, throughput = requests/sec. Batching throughput badhata par latency trade-off. Application ke hisaab se balance."

---

## 3. Availability & Reliability (HA, SLA, SLO)

**Availability** = system kitna time up rehta, aksar "nines" me (99.9% = 8.7 hrs downtime/year, 99.99% = 52 min/year). **Reliability** = system sahi kaam karta bina fail hue. High Availability (HA) achieve karne ke liye **redundancy** chahiye — koi single component fail ho to backup le le. Iske liye multiple replicas, multiple availability zones, load balancers, failover mechanisms lagte. **SLA** (Service Level Agreement) = customer se promise (99.9% uptime), **SLO** (Objective) = internal target, **SLI** (Indicator) = actual measured metric. Architect ko yeh define aur design karna padta.

ML me yeh critical — agar tera LLM endpoint down ho gaya, poori application ruk jaati. Isliye multi-AZ deployment, health checks, auto-restart, aur fallback (jaise primary model down to secondary/cached response) design karte.

**Networking analogy:** Yeh tera ghar ka maidan — HSRP/VRRP redundancy, dual uplinks, multi-site failover. SLA/SLO tune networking me jiya hai. ML me same principle — bas components ML hain.

**Interview line:** "HA via redundancy — multi-AZ replicas, health checks, failover. SLA/SLO/SLI define karke measure. ML endpoint ke liye multi-AZ + fallback critical."

---

## 4. Consistency Models (CAP Theorem)

Distributed systems me ek fundamental trade-off hai — **CAP theorem**: Consistency (sab nodes same data dekhein), Availability (har request ko response mile), Partition tolerance (network toot-phoot me bhi chale). Theorem kehta hai teeno ek saath fully nahi mil sakte — network partition hone pe tumhe C ya A me se ek chunna padta. Practical systems **eventual consistency** (thodi der baad sab sync ho jaayenge — high availability) ya **strong consistency** (hamesha latest data — par thodi availability/latency cost) chunte.

ML me yeh feature stores aur model registries me aata. Jaise feature store me — real-time features ko strong consistency chahiye (galat feature = galat prediction), par experiment metadata me eventual consistency chalega.

**Networking analogy:** Eventual consistency = BGP convergence — route change propagate hone me thoda time, par eventually sab consistent. Strong consistency = synchronous replication (turant, par slow).

**Interview line:** "CAP — partition me C ya A chunna. Real-time features strong consistency, metadata eventual. Trade-off latency/availability vs freshness."

---

## 5. Load Balancing

Jab horizontal scale karte, load balancer traffic ko multiple servers me distribute karta. Algorithms: round-robin (baari-baari), least-connections (jise kam load), weighted (capacity ke hisaab se), consistent hashing (same client same server — caching ke liye useful). Load balancer health checks bhi karta — jo server down, usko traffic nahi bhejta (automatic failover). ML serving me yeh critical — kai inference servers ke aage LB, taaki load evenly bant jaaye aur ek server marne pe baaki handle karein.

**Networking analogy:** Yeh literally tera domain — LB, ECMP, traffic distribution, health-based routing. Tu isko sabse achhe samjhta interview panel se.

**Interview line:** "LB traffic distribute (round-robin/least-conn/weighted) + health checks for failover. ML serving me inference servers ke aage LB for scale + resilience."

---

## 6. Caching

Caching matlab frequently-accessed data ko fast storage (memory) me rakhna taaki baar-baar slow source (DB/compute) na hit karna pade. Yeh latency dramatically kam karta aur backend load ghatata. Levels: client-side, CDN (edge), application cache (Redis/Memcached), DB cache. Key concepts: **cache hit/miss** (data mila ya nahi), **eviction policy** (jagah bharne pe kya hataao — LRU sabse common), **TTL** (kitni der valid), **cache invalidation** (data badle to cache update — "hardest problem in CS").

ML me caching bahut useful — LLM me same prompt ka response cache karo (repeated queries pe LLM call bachao = paisa + latency), embedding cache, retrieval results cache. Yeh cost optimization ka bada lever hai (LLM calls mehenge).

**Networking analogy:** Caching = DNS caching ya CDN edge caching jo tu jaanta. Repeated lookups ko local se serve, origin ko bachao.

**Interview line:** "Cache frequently-accessed data in memory (Redis) — latency down, backend load down. LLM me prompt/response caching = big cost saver. LRU eviction, TTL, invalidation manage."

---

## 7. Message Queues & Async Processing

Kabhi kaam turant nahi karna hota (synchronous), balki background me (asynchronous). Message queue (SQS, Kafka, RabbitMQ) producer aur consumer ke beech buffer hota — producer message daal deta, consumer apni speed se process karta. Yeh **decoupling** deta (components independent), **load smoothing** (traffic spike absorb), aur **reliability** (consumer down to messages queue me safe). ML me batch inference, training jobs, document ingestion pipelines async hote — request aaya, queue me daala, background worker process karta, result baad me.

**Networking analogy:** Queue = buffer/QoS queue jaise — burst traffic ko absorb karke steady rate pe process. Decoupling = loosely coupled network segments.

**Interview line:** "Queue (SQS/Kafka) producer-consumer decouple, load smooth, reliability. ML me batch inference/training/ingestion async — spike absorb, fault tolerance."

---

## 8. Database Scaling

Jab data/traffic badhta, single DB tootta. Techniques: **Replication** (copies banao — read replicas se read load bant jaata, HA bhi), **Sharding/Partitioning** (data ko tukdon me alag DBs pe — har shard ek subset, write scale hota), **Indexing** (fast queries), **SQL vs NoSQL choice** (structured+transactions = SQL/PostgreSQL; scale+flexible = NoSQL/DynamoDB). ML me — metadata SQL me (MLflow backend PostgreSQL), large artifacts object store (S3), vectors specialized vector DB me.

**Networking analogy:** Replication = redundant config sync across devices. Sharding = load-splitting across parallel paths. Read replica = anycast read nodes.

**Interview line:** "Replication for read-scale + HA, sharding for write-scale, indexing for query speed. SQL for transactions, NoSQL for scale. ML: metadata SQL, artifacts S3, embeddings vector DB."

---

## 9. Storage Types (ML me important)

Teen tarah ka storage samajhna: **Block storage** (EBS — ek instance ko attached disk, fast, DBs ke liye), **Object storage** (S3 — unlimited, cheap, durable, HTTP access, ML artifacts/datasets/models ideal), **File storage** (EFS — shared filesystem, multiple instances mount karein, shared datasets). ML me S3 backbone hai — datasets, model artifacts, checkpoints sab wahan; training instances EBS/EFS use karte.

**Networking analogy:** Object storage = distributed file server accessible over network (HTTP). Block = local disk. File = NFS share.

**Interview line:** "Block (EBS) fast local, Object (S3) cheap+scalable+durable for ML artifacts/data, File (EFS) shared. S3 = ML storage backbone."

---

## Interview Q&A (Foundations)

**Q: Vertical vs horizontal scaling?** — "Vertical = bigger machine (limit + SPOF). Horizontal = more machines + LB (unlimited scale, fault-tolerant, par coordination complexity). Modern/ML systems horizontal prefer."

**Q: Latency vs throughput?** — "Latency = single request speed, throughput = requests/sec. Batching throughput badhata, latency trade-off."

**Q: CAP theorem?** — "Partition me Consistency ya Availability chunna. Real-time features strong, metadata eventual."

**Q: Caching kyun ML me?** — "LLM prompt/response cache = repeated queries pe compute+cost bachao. Latency down. Big cost lever."

**Q: 99.9% availability kaise?** — "Multi-AZ redundancy, LB + health checks, auto-failover, fallback. SLA/SLO define + measure via SLI."

---

## 🎓 Deep Dive & Q&A (Teacher Session — zero-se-expert, section-wise)

> Live teaching ka nichod — jo sawaal/confusion clear hue, why-level depth. Doc ke paragraph points ka reflex-banane wala version.

### Section 1 — Scalability (Vertical vs Horizontal)

**Vertical (scale UP) — machine badi karo:**
- Ek server ko powerful (zyada CPU/RAM/GPU).
- ➕ Simple (no code change). ➖ Physical limit, mehenga (exponential), SPOF.

**Horizontal (scale OUT) — machines add karo:**
- Aur servers, load LB se baanto.
- ➕ Unlimited scale, fault-tolerant. ➖ Coordination/consistency complexity.
- Modern/ML systems horizontal prefer (scale + resilience).

**Nuance:** Sab horizontal nahi ho sakta — stateful (jaise 70B model jo ek GPU me fit nahi) pehle thoda vertical (bada GPU) phir horizontal (multi-GPU/replicas). Practice = **mix**.

**CRITICAL — Spike ko autoscale se solve karo, permanent bade hardware se NAHI:**
- **Spiky/time-based load → horizontal autoscaling** (peak pe servers add, off-peak scale-in). Sirf peak ke ghante ka paisa.
- **Sustained high → horizontal replicas** (LB peeche).
- **Vertical** sirf jab baseline pe hi ek unit under-powered (bada model, GPU tight), na ki temporary spike ke liye.

**⭐ GPU billing insight (yaad rakh — bahut common confusion):**
- GPU billing = **ALLOCATED (running) time pe, utilization pe NAHI.**
- GPU 100% use ya 5% idle → **same bill.** Instance launch = reserved = poora kiraya (chahe use ho ya na).
- Idle allocated GPU = wasted money (30% util = 70% waste).
- Isliye permanent bada GPU for peak-only need = 24×7 bill for 4hr use = waste. **Autoscale** = sirf zaroorat ke ghante ka paisa.
- Analogy: reserved parking spot (book kiya toh kiraya poora, car khadi ho ya na) / committed fat pipe vs burstable bandwidth.

**Networking analogy:** Vertical = router ko bigger chassis se replace (limit aayegi). Horizontal = kai routers/links load-share (ECMP). Spike = burstable bandwidth (autoscale), na ki permanent over-provisioned fat pipe.

**Interview one-liner:**
> "Vertical = bigger machine (limit + SPOF + costly). Horizontal = more machines + LB (unlimited + fault-tolerant, coordination complexity). Modern/ML horizontal prefer. Spike → autoscaling (peak add, off-peak remove — cost-efficient), kyunki GPU billing allocated-time pe hoti utilization pe nahi — idle GPU bhi poora bill."

**Q&A hue:**
- **Q: LLM inference slow, vertical ya horizontal?** → Diagnose pehle (spike ya sustained?). Spike → horizontal autoscaling. Sustained → horizontal replicas. Vertical sirf baseline under-powered. Limitation: coordination complexity + cost vs criticality.
- **Q: Peak-hour spike → vertical ya autoscaling?** → Horizontal autoscaling — temporary load permanent hardware se solve nahi karte; idle GPU bhi poora bill khaata.

### Section 2 — Latency vs Throughput

**Latency** = ek single request kitni jaldi complete (200ms). **User feel karta.**
**Throughput** = system per second kitne requests handle kare (1000 QPS). **System capacity.**

**Trade-off (highway analogy):** latency = ek car ka A→B time; throughput = per hour kitni car. Zyada car bharo (throughput up) → traffic → har car slow (latency up).

**ML example:** Batching → throughput badhta (GPU bhar ke chalta) par individual latency badhti (batch-fill wait).

**⭐ DECISION RULE (yeh reflex banao — yahan slip hua tha):**
- **User wait kar raha (real-time chat)** → **LATENCY** priority
- **Koi wait nahi (bulk/background/batch/nightly job)** → **THROUGHPUT** priority
- GP-02 se consistent: training = throughput, inference = latency.
- Common trap: "1 lakh docs jaldi chahiye" ≠ latency. Koi user wait nahi → individual doc time irrelevant → **throughput** (batching se total jaldi).

**LLM nuance (senior signal):** do latency — **first-token latency** (pehla word — perceived speed) vs **total latency** (poora jawab). Streaming se first-token jaldi dikhate → perceived latency kam.

**Networking analogy:** Latency = packet RTT (ping). Throughput = bandwidth (Gbps). Real-time voice = latency; nightly bulk transfer/backup = throughput/bandwidth (packet RTT irrelevant). Fat pipe high-throughput par high-latency ho sakta (satellite) — dono alag optimization.

**Interview one-liner:**
> "Latency = single-request speed, throughput = requests/sec. Batching throughput badhata, latency trade-off. User waiting (chat) = latency-priority; background bulk (nightly job) = throughput-priority. Packet RTT vs bandwidth jaisa."

**Q&A hue:**
- **Q: Nightly job 1 lakh docs summarize (no user waiting) — latency ya throughput?** → **Throughput** (koi wait nahi, individual doc time irrelevant, total batch jaldi). Technique: batching. = bulk file transfer/backup networking.
- **Q: live chat vs nightly bulk?** → chat = latency, bulk = throughput.

### Section 3 — Availability & Reliability (HA, SLA/SLO/SLI) [tera maidan]

**Availability** = uptime, "nines" me:
- 99.9% = ~8.7 hr/yr down | 99.99% = ~52 min/yr | 99.999% = ~5 min/yr (har nine exponentially costly).

**Reliability** = up **aur sahi** kaam (availability = up; reliability = up + correct).

**⭐ HA ke 3 PILLAR (yeh reflex — "backup hai" adhoora):**
1. **Redundancy** — multi-AZ replicas (ek zone gire, doosra serve). Multiple instances.
2. **Detection + Failover** — health checks (dead instance detect) → auto-switch to healthy. Auto-restart crashed pods. (Bina detection ke backup bekaar.)
3. **Fallback / Graceful Degradation** — jab primary/model down: secondary (chhota/sasta) model, ya cached response, ya "abhi busy, retry" message. Poora fail na ho.

**SLA / SLO / SLI:**
- **SLA** (Agreement) = customer se promise/contract ("99.9% warna refund")
- **SLO** (Objective) = internal target (aksar SLA se strict — buffer)
- **SLI** (Indicator) = actual measured ("is mahine 99.93% mila")
- Soch: SLI = mila, SLO = chahte the, SLA = promise kiya.

**ML me:** LLM endpoint down → poori app ruk. Multi-AZ + health check + auto-restart + fallback (secondary/cached) critical. 99.99% ke liye kabhi multi-REGION bhi (poora region gira toh).

**Networking analogy (EXACT maidan):** HSRP/VRRP (gateway redundancy), dual uplinks, multi-site failover, BFD (dead-path detect + reroute), uptime SLA/MTTR. ML me same — components ML.

**Interview one-liner:**
> "Availability = uptime (99.99% = 52min/yr). HA 3 pillar: redundancy (multi-AZ replicas), detection+failover (health checks → auto-switch, auto-restart), fallback (secondary model/cached/graceful). SLA (promise) > SLO (target) > SLI (measured). LLM endpoint multi-AZ + fallback. HSRP/dual-uplink/BFD jaisa."

**Q&A hue:**
- **Q: 99.99% LLM chatbot — 3 design + fallback?** → (1) multi-AZ replicas, (2) LB + health checks + auto-failover + auto-restart, (3) fallback: secondary model / cached / graceful degradation. + multi-region for extreme. Analogy: HSRP + dual uplink + BFD.
- **Q: HA ke 3 pillar?** → Redundancy, Detection+Failover, Fallback/Degradation.

### Section 4 — Consistency & CAP Theorem

**Setup:** Data multiple nodes pe replicate → "sab copies pe same data hai?" Ek node pe likha, doosre se turant pado → latest milega ya purana?

**CAP — 3 cheezein, teeno saath NAHI:**
- **C (Consistency):** har read ko latest data (sab nodes same).
- **A (Availability):** har request ko response mile (chahe nodes down).
- **P (Partition tolerance):** network toot-phoot me bhi chale.

**Theorem:** Partition (network split) hone pe **C ya A chunna padta** (P distributed me zaroori). Kyun? Link toota, B tak update nahi pahuncha:
- **C chuno:** B response nahi deta (galat data se accha koi nahi) → consistent, not available.
- **A chuno:** B purana data de deta → available, inconsistent (stale).

**2 practical models:**
- **Strong consistency:** hamesha latest (galat kabhi nahi). Slow (sync), partition me availability giri.
- **Eventual consistency:** thodi der baad sab sync. Turant stale mil sakta, par fast + available.

**⭐ ML me kahan kya:**
- **Strong** → real-time critical features (fraud detection me balance — stale = galat decision = paisa loss).
- **Eventual** → dashboard analytics, experiment metadata, model registry listing, historical counts (thodi der purana OK).
- **Nuance:** ek hi system me feature-wise mix — critical feature strong, non-critical eventual.

**Networking analogy (EXACT domain):**
- **Eventual = BGP convergence** — route change propagate hone me time, kuch routers temporarily purana, eventually sab consistent.
- **Strong = synchronous replication** — turant sab same, par slow (sabka ack).
- **Partition = network split** — halves ek doosre ko nahi dekhte; purane pe chalein (available) ya rukein (consistent).

**Interview one-liner:**
> "CAP — partition me C ya A, dono nahi. Strong = always latest (slow, A giri in partition); eventual = fast/available par stale temporarily. ML: real-time features strong (fraud/balance — stale=galat decision), metadata/analytics/logs eventual. BGP convergence = eventual, sync replication = strong."

**Q&A hue:**
- **Q: Fraud detection balance feature — strong ya eventual?** → Strong (stale balance = galat approve = loss). Eventual chalega: dashboard/metadata/historical counts. Feature-wise mix possible. BGP = eventual, sync-replication = strong.

### Section 5 — Load Balancing [tera EXACT maidan]

**Problem:** Horizontal scale → 5 inference servers. Kaunsa request kaunse server pe? LB traffic baanta (warna ek pe load, baaki khali).

**Algorithms:**
- **Round-robin** — baari-baari. Simple, equal-capacity ke liye. **Problem:** uneven request duration me lambi requests ek server pe jam.
- **Least-connections** — kam active load wale server pe. **Uneven request DURATION ka solution** (LLM: kuch 100ms, kuch 10sec).
- **Weighted** — capacity ke hisaab se. **Uneven SERVER CAPACITY ka solution** (mixed hardware).
- **Consistent hashing** — same client → same server. Cache affinity ke liye.

**⭐ Least-conn vs Weighted (farak crisp rakho):**
- Least-connections = requests ki duration uneven → load-aware.
- Weighted = servers ki capacity uneven → capacity-aware.
- Dono alag problem; LLM uneven-duration scenario me primary = **least-connections**.

**Health checks:** LB server ko probe karta; unhealthy → traffic band (auto-failover). Yeh HA ka detection+failover pillar.

**ML nuance:** LLM requests uneven (chhota jawab vs 2000-token generation) → round-robin se jam → **least-connections/load-aware** behtar.

**Networking analogy (EXACT domain):** ECMP (multi-path distribute), least-conn = load-aware routing, consistent hashing = flow affinity/stickiness (stateful firewall), health check = BFD/probe (dead-path detect + reroute).

**Interview one-liner:**
> "LB distributes (round-robin/least-conn/weighted/consistent-hashing) + health checks for failover. LLM uneven request durations → least-connections/load-aware (round-robin jam kar dega). Weighted for mixed capacity, consistent-hashing for cache affinity. ECMP + health-based routing jaisa."

**Q&A hue:**
- **Q: LLM requests uneven (100ms vs 10sec) — round-robin ya kuch aur?** → Least-connections/load-aware (round-robin lambi requests jam kar deta). Weighted tab jab servers bhi mixed-capacity. Networking: ECMP + load-aware routing + flow affinity.
