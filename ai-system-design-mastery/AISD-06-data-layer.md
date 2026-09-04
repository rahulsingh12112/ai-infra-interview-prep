# AISD-06: Data Layer (RDS + Vector DB + S3/EFS/EBS)

> Station 6 — Data kahan store karna hai. Golden rule: data type + size decides.

## Paragraph (samajh)
AI me teen tarah ka data: structured metadata (rows/cols) → SQL (RDS), badi files (GBs, model/dataset/checkpoint) → S3 (object store, kabhi DB nahi), embeddings (RAG vectors) → Vector DB (similarity search). Galat jagah = common interview galti. Golden rule: data ka TYPE aur SIZE decide karta store, "sab DB me" galat.

## Points

**1. RDS PostgreSQL — structured metadata (SQL)**
Experiment runs, params, metrics, user data. Rows/cols. MLflow backend. Strong consistency (fraud features). Read-heavy → replicas.

**2. S3 (Object Store) — badi files (ML backbone)**
Model files (GBs), datasets, checkpoints, backups. KABHI DB me nahi — bada binary DB me = performance kill (queries/backup/replication slow). S3 = bade objects ke liye designed, unlimited, durable (11 nines). DB me sirf path store. **Primary reason: right tool (designed for objects). Sasta = bonus.**

**3. Vector DB — embeddings (RAG)**
Documents → chunked → embedded → vectors. Similarity search (nearest neighbors) chahiye — normal DB slow. OpenSearch/pgvector/Pinecone. Sharded + replicated.

## Storage types (quick ref)
```
EBS  = ek instance ki personal fast disk (DB data, OS)
S3   = sabke liye unlimited sasta bucket (ML backbone)
EFS  = kai instances ka shared drive (shared datasets mount)
```

## Diagram
```
DATA TYPE             → STORE                    → KYUN
Structured metadata   → 🗃️ RDS (SQL)             → rows/cols, MLflow, transactions
Badi files (GBs)      → 🪣 S3 (objects)           → designed for objects, never DB
Embeddings/vectors    → 🔎 Vector DB              → similarity search, sharded
```

## Interview one-liner
> "Type+size decides: metadata → SQL/RDS (structured, MLflow, replicas), big files → S3 (designed for objects, never DB — sirf path DB me), embeddings → Vector DB (similarity search, sharded). EBS=local, S3=backbone, EFS=shared."

## Practiced Q — 5GB model, MLflow metrics, 10M embeddings
(a) 5GB model → **S3** (badi binary, DB me kabhi nahi — perf kill. S3 designed for objects.)
(b) MLflow metrics → **RDS** (structured, SQL, read-replicas, MLflow backend)
(c) 10M embeddings → **Vector DB** (similarity search — OpenSearch/pgvector/Pinecone, sharded+replicated)

⚠️ "S3 kyunki sasta" galat primary reason. Sahi: "S3 kyunki badi binary ke liye designed. DB me = performance kill. Sasta = bonus."

⭐ Key line: "Badi binary → S3 (designed for objects, DB kabhi nahi). Structured → SQL. Vectors → Vector DB. Right tool, not default-to-DB."
