# MongoDB Interview Questions 

## 1. Explain MongoDB’s document-oriented data model.

MongoDB stores data as **BSON documents** (JSON-like) — self-contained, hierarchical records with flexible key-value pairs, nested objects, and arrays.  
- No fixed schema → fields can vary per document  
- Documents grouped into **collections** (like tables, but schema-free)  
- Encourages denormalization / embedding for read-heavy access patterns  
→ Ideal for semi-structured / evolving data (e.g., user profiles, catalogs, IoT events, content management).

## 2. How do documents and collections differ from rows and tables?

- **Document** ≈ Row: Single record, but hierarchical (nested fields, arrays) instead of flat columns. Can have different fields per document.  
- **Collection** ≈ Table: Group of documents, but schema-less (no enforced structure across documents).  
Key differences:  
- No rigid columns → dynamic schema  
- No mandatory joins (prefer embedding)  
- Documents have 16 MB size limit (BSON max)  
- Collections can hold billions of documents; sharded for scale

## 3. What is BSON, and why does MongoDB use it?

**BSON** = Binary JSON — extended binary-encoded serialization of JSON-like documents.  
Adds:  
- Type information (e.g., Date, ObjectId, Binary, Decimal128)  
- Efficient storage & traversal (length prefixes, no string parsing)  
- Support for binary data, regex, timestamps with ms precision  

MongoDB uses BSON because:  
- Faster parsing & indexing than pure JSON  
- Preserves rich types (especially numbers & dates)  
- Enables efficient queries, sorting, and aggregation

## 4. How does indexing work in MongoDB?

Indexes speed up queries by storing sorted pointers to data.  
- **Default**: `_id` index (unique, hashed or ascending)  
- Types: single-field, compound, multikey (arrays), text, hashed, 2d/2dsphere (geo), TTL, partial, sparse, clustered (8.0+), vector (Atlas/Vector Search)  
- Storage: B-tree (most), WiredTiger LSM for some cases  
- Impact: Fast reads (O(log n)), but writes slower (index maintenance)  
- Best practice: Create indexes covering common sort/filter/project patterns; use compound indexes wisely (equality → sort → range rule)

## 5. What is the aggregation framework in MongoDB?

Powerful pipeline for data processing & transformation — MongoDB’s equivalent to SQL GROUP BY + JOIN + window functions + more.  
Syntax: array of stages (`$match`, `$group`, `$lookup`, `$unwind`, `$sort`, `$project`, `$addFields`, `$merge`, `$out`, `$vectorSearch` in 2025–2026).  
Use cases:  
- Analytics, reporting, faceted search  
- Joining collections (`$lookup`)  
- Real-time transformations  
- Vector similarity search (Atlas)  
Runs on server → efficient, avoids client-side processing

## 6. Explain the difference between embedding and referencing.

**Embedding** (denormalization):  
- Store related data inside the parent document (nested object/array)  
- Pros: Single read, atomic updates, fast  
- Cons: Document size limit (16 MB), duplication, hard to update shared data  

**Referencing** (normalized):  
- Store DBRef or manual reference (e.g., ObjectId) to related document  
- Pros: No duplication, easy updates to shared data  
- Cons: Requires `$lookup` or multiple queries → slower reads  

Rule of thumb (2026): Embed for 1:1, 1:few, read-heavy; reference for 1:many (many = hundreds+), write-heavy shared entities.

## 7. How does MongoDB handle relationships between collections?

No foreign keys or enforced referential integrity.  
Options:  
1. **Embedding** — preferred for performance (one read)  
2. **Referencing** + manual joins via `$lookup` (aggregation) or client-side  
3. **Bucketing** / **subset** patterns for large 1:many  
4. **Computed / materialized views** (via `$merge`, change streams + triggers)  
5. **Application-level enforcement** (common in practice)  

MongoDB pushes denormalization to avoid joins whenever possible.

## 8. What is replication in MongoDB?

**Replication** = maintaining multiple copies of data across nodes for **high availability**, **fault tolerance**, and **read scaling**.  
Primary (writes) + secondaries (replicate via oplog).  
Automatic failover: if primary fails, eligible secondary elects new primary.  
Supports: read preference (secondary reads), delayed members, arbiter (for voting).

## 9. What is a replica set, and why is it important?

**Replica set** = group of `mongod` processes maintaining the same dataset (1 primary + 0–50 secondaries + optional arbiters).  
Importance:  
- High availability (automatic failover ~seconds)  
- Data redundancy & durability  
- Read scaling (secondaries)  
- Basis for sharded clusters (each shard is a replica set)  
Minimum recommended: 3 members (odd number for voting)

## 10. How do MongoDB support multi-document transactions?

Since 4.0 (replica sets), 4.2 (sharded clusters): **ACID multi-document / multi-collection / multi-database transactions**.  
- `session.withTransaction(async () => { ... })` in drivers  
- Snapshot isolation  
- All-or-nothing commit/rollback  
- Distributed two-phase commit (sharded)  
Trade-off: Higher latency & locking → use only when needed (financial, inventory); prefer single-doc atomicity otherwise.

## 11. What is sharding in MongoDB, and how does it improve scalability?

**Sharding** = horizontal partitioning — splits collection into chunks distributed across shards (each shard = replica set).  
Components: mongos (router), config servers, shard servers.  
Shard key: chooses how data splits (hashed for even distribution, ranged for locality).  
Improves:  
- Storage capacity (beyond single node)  
- Throughput (parallel writes/reads)  
- Linear scale-out  
With distributed transactions & zoned sharding (2020s enhancements) — very powerful for massive workloads.

## 12. What are capped collections?

Fixed-size collections that overwrite oldest documents when full (FIFO queue behavior).  
Features:  
- High-performance inserts (no index updates on removal)  
- Preserve insertion order (natural order = insertion order)  
- TTL-like but size-based  
Use cases: logging, caching, time-series data before time-series collections existed (now prefer time-series).

## 13. How does MongoDB ensure data durability?

- **Journaling** (WiredTiger): write-ahead log → crash recovery  
- **Write Concern**: `{ w: "majority" }` — acknowledged only after majority of replica set writes  
- **Journal + Fsync** options  
- **Replication** — data survives node failure  
- **Change Streams** + oplog for downstream durability  
Default: durable on majority commit (since ~4.0+ defaults).

## 14. What is the difference between find() and aggregation pipelines?

- **find()**: Simple query + projection + sort + limit/skip. Fast for basic CRUD, uses indexes directly.  
- **Aggregation pipeline**: Multi-stage processing (`$match` → `$group` → `$lookup` → … → `$out`).  
  - Can join, reshape, compute, bucket, vector search  
  - Server-side execution → efficient for complex analytics  
  - More flexible but heavier (overhead)  

Use `find()` for lookups; aggregation for transformations, reporting, joins.

## 15. What are common schema design mistakes in MongoDB?

1. **Unbounded arrays** → documents hit 16 MB limit → massive slowdowns  
2. **Over-normalizing** (too many references + lookups everywhere) → relational anti-pattern in NoSQL  
3. **Ignoring access patterns** → designing like SQL instead of query-first  
4. **Poor shard key choice** → jumbo chunks, hot shards  
5. **No / bad indexes** → collection scans at scale  
6. **Treating it like RDBMS** → forcing joins instead of embedding  
7. **Dynamic field type changes** → breaks indexes / queries  
8. **Forgetting eventual consistency** in sharded / replicated setups  
9. **Massive documents without subset / bucketing** patterns  
10. **No schema validation** (MongoDB supports JSON Schema) → garbage data


