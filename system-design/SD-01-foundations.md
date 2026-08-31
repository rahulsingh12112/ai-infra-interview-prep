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
