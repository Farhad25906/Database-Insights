# Scenario-Based Database Interview Questions

This document contains a collection of real-world database design scenarios, performance troubleshooting guides, and architectural trade-offs.

## 🧭 Table of Contents

- [1. Banking System Consistency](#1-banking-system-consistency)
- [2. High-Volume Write-Heavy Logs](#2-high-volume-write-heavy-logs)
- [3. Troubleshooting Slow MongoDB Queries](#3-troubleshooting-slow-mongodb-queries)
- [4. Modeling Many-to-Many in MongoDB](#4-modeling-many-to-many-in-mongodb)
- [5. Prisma Transaction Rollbacks](#5-prisma-transaction-rollbacks)
- [6. Zero-Downtime MySQL Migration](#6-zero-downtime-mysql-migration)
- [7. Optimizing Relational Joins](#7-optimizing-relational-joins)
- [8. SQL vs NoSQL for Evolving Schemas](#8-sql-vs-nosql-for-evolving-schemas)
- [9. Rollback Safety in Order Management](#9-rollback-safety-in-order-management)
- [10. RBAC at the Data Layer with Prisma](#10-rbac-at-the-data-layer-with-prisma)
- [11. Scaling MongoDB with Sharding](#11-scaling-mongodb-with-sharding)
- [12. Relational Integrity vs JSON Flexibility](#12-relational-integrity-vs-json-flexibility)
- [13. Debugging Production Deadlocks](#13-debugging-production-deadlocks)
- [14. Solving N+1 Query Problems](#14-solving-n1-query-problems)
- [15. Redesigning Monolithic Databases](#15-redesigning-monolithic-databases)

---

## 1. Banking System Consistency
**Scenario:** You are building a banking system requiring strict consistency. Which database type would you choose and why?

### Answer:
I would choose a relational database with strong ACID guarantees — most likely **PostgreSQL**.

Banking requires strict consistency to prevent double-spending, overdrafts, or incorrect balances. PostgreSQL gives me full ACID compliance, mature MVCC, serializable isolation level when needed, excellent support for constraints, foreign keys, CHECK constraints, and partial indexes. It also has very good performance for transactional workloads when properly indexed and partitioned.

I would avoid eventually-consistent NoSQL databases (Cassandra, DynamoDB base models, MongoDB without strong majority writes) for core accounts and transactions. For non-critical parts (audit logs, notifications) I might use complementary systems, but the money movement stays in Postgres with serializable isolation or careful application-level checks + pessimistic locking where appropriate.

---

## 2. High-Volume Write-Heavy Logs
**Scenario:** A system needs to handle millions of write-heavy logs per day. How would you design the database?

### Answer:
For high-volume, write-heavy, append-only logs I would choose a purpose-built time-series or log-oriented database rather than a general-purpose relational one. My preferred stack would be:

- **ClickHouse** (columnar, extremely fast inserts & aggregations)
- **TimescaleDB** (PostgreSQL extension — good if I want to stay in SQL ecosystem)
- **Elasticsearch / OpenSearch** (if full-text search + logs/analytics are important)
- **Kafka → S3 / Parquet + Athena or BigQuery** for long-term cheap storage

**Typical design:**
1. Partition by date / tenant / service
2. Minimal schema (timestamp, level, service, message, context JSONB / object)
3. Write directly or via Kafka for buffering & ordering
4. Compression + partitioning to keep insert throughput high
5. Materialized views or roll-ups for frequent dashboards
6. Retention policy (hot 30 days → cold S3 / cheaper storage)

Relational databases (even PostgreSQL) become expensive and slow at 10M+ inserts/day without very careful partitioning and index strategy.

---

## 3. Troubleshooting Slow MongoDB Queries
**Scenario:** Your MongoDB queries are slow despite indexes. How would you troubleshoot?

### Answer:
Step-by-step troubleshooting approach:

1. **Explain the query** — use `.explain("executionStats")` or Atlas profiler
   - Look at: `winningPlan` vs `rejectedPlans`, `totalDocsExamined`, `keysExamined`, stage (`COLLSCAN` vs `IXSCAN`), `nReturned` vs `examined` ratio
2. **Check index coverage**
   - Is the index actually being used?
   - Are equality fields first, then sort fields, then range?
   - Compound index order correct? (**ESR rule**: Equality → Sort → Range)
3. **Look for large documents or array scans** — unwinding large arrays kills performance
4. **Check working set** — if indexes + frequently accessed data don’t fit in RAM → page faults
5. **Review query shapes & cardinality** — very low selectivity filters still scan many documents
6. **Profile slow queries** in Atlas / `mongostat` / `mongotop` / profiler collection
7. **Consider covered queries**, projection to reduce document size, collation strength issues

Most common root causes: wrong compound index order, missing sort index, huge `$in` / `$nin` arrays, or regex without index prefix.

---

## 4. Modeling Many-to-Many in MongoDB
**Scenario:** You need to model a many-to-many relationship in MongoDB. How would you approach it?

### Answer:
Three main patterns — choice depends on access pattern and cardinality:

**A. Array of references (most common)**
```javascript
// Users
{ _id: "...", name: "Alice", followedBy: [ObjectId("..."), ObjectId("...")] }

// Posts
{ _id: "...", content: "...", likedBy: [ObjectId("...")] }
```
- Good when one side has limited cardinality (e.g., user follows < 5000 accounts).

**B. Array of sub-documents (when data is small & read-heavy)**
```javascript
{ 
  _id: "...",
  tags: [
    { tagId: ObjectId("..."), name: "javascript", addedAt: ISODate() }
  ]
}
```
- Fast reads, but updates require `arrayFilters` or large document rewrites.

**C. Join collection (normalized — when both sides can be very large)**
```javascript
// follows collection
{ followerId: ObjectId, followeeId: ObjectId, createdAt: ISODate }
```
- Use compound index on `{followerId:1, createdAt:-1}`.
- Good for very high cardinality or when both directions are queried equally.

---

## 5. Prisma Transaction Rollbacks
**Scenario:** A Prisma transaction partially fails during a payment flow. What should happen and why?

### Answer:
The entire transaction should rollback and no partial state should be committed.

**Why:**
- Payment flows are classic financial transactions → partial execution (e.g. money deducted but order not created) breaks business invariants.
- Prisma’s `$transaction` API is atomic — if any operation throws, the whole block is rolled back.
- Wrap the whole flow in one `$transaction` and handle errors with specific catch blocks.
- **Extra safety:** Log the full failure context, return user-friendly message, and ideally put the payment in a retry queue if the failure was transient.

---

## 6. Zero-Downtime MySQL Migration
**Scenario:** You need to migrate a large MySQL database with zero downtime. What strategies would you use?

### Answer:
Proven zero-downtime strategies:

- **gh-ost (GitHub)**: Trigger-based, no read locks. My #1 choice for most schema changes.
- **pt-online-schema-change (Percona)**: Battle-tested, but uses triggers.
- **Dual write + backfill**:
  1. Write to old & new schema simultaneously.
  2. Backfill historical data.
  3. Switch reads → new table.
  4. Clean up old table.
- **Logical replication + rename swap**:
  1. Create new database / schema.
  2. Replicate changes with binlog.
  3. When caught up → cutover (rename tables).

**Key precautions:** Careful FK handling, testing on staging with real traffic, and having an instant rollback plan.

---

## 7. Optimizing Relational Joins
**Scenario:** A relational query with multiple joins is slow. How would you optimize it?

### Answer:
Structured optimization checklist:

1. **EXPLAIN ANALYZE**: Identify full scans, bad join order, or temp tables.
2. **Indexing**: Add composite indexes matching `WHERE` + `JOIN` + `ORDER BY`.
3. **Early Filtering**: Push filters into subqueries/CTEs; use `EXISTS` instead of `IN`.
4. **Selective Denormalization**: Cache counts or timestamps in the parent table.
5. **Materialized Views**: Use for expensive aggregations in PostgreSQL.
6. **Query Splitting**: Sometimes fetching the main entity first, then batching relations is faster.
7. **Aggressive Caching**: Use Redis for expensive read patterns.

---

## 8. SQL vs NoSQL for Evolving Schemas
**Scenario:** You are designing a schema that frequently changes. Would you choose SQL or NoSQL?

### Answer:
I would lean toward **NoSQL (MongoDB)** or **PostgreSQL with JSONB**, depending on consistency needs.

**Why NoSQL/JSONB:**
- Rapid iteration without costly migrations.
- Schema-on-read allows teams to move fast.
- Good support for evolving nested documents.

**Choose PostgreSQL + JSONB when:**
- Strong consistency/transactions are still required.
- Complex reporting/analytics are important (SQL ecosystem is better).
- You want type safety in the application (Prisma + JSONB).

---

## 9. Rollback Safety in Order Management
**Scenario:** You need to ensure rollback safety in an order management system. How would you design it?

### Answer:
Design principles for safe order processing:

- **Atomic Transactions**: Single transaction for money movement + order state.
- **Pessimistic Locking**: Use `SELECT FOR UPDATE` on inventory rows.
- **Sagas/Compensating Transactions**: For microservices (Reserve → Pay → Confirm; rollback on fail).
- **Outbox Pattern**: Write order events to an outbox table in the same transaction.
- **State Machine**: Strict validation for state transitions (PENDING → PAID → SHIPPED).
- **Idempotency Keys**: Critical for payment and external API calls.

---

## 10. RBAC at the Data Layer with Prisma
**Scenario:** You must enforce role-based access control at the data layer. How would Prisma fit into this?

### Answer:
Prisma doesn't have built-in Row-Level Security (RLS), so we use two main approaches:

**1. PostgreSQL RLS + Prisma Raw Queries**
- Define RLS policies in Postgres.
- Set session variables (e.g., `current_user_id`) via `SET LOCAL`.
- Prisma executes queries under that context.

**2. Application-Level Middleware (Most Common)**
```typescript
const canAccessUser = (user: User, actor: Actor) =>
  actor.id === user.id || actor.roles.includes("ADMIN");

const posts = await prisma.post.findMany({
  where: {
    OR: [
      { authorId: actor.id },
      { visibility: "PUBLIC" },
      // more complex conditions
    ]
  }
});
```
- Use Prisma Middleware or Extensions to inject `where` filters automatically (e.g., `tenantId` or `userId`).
- Use libraries like **CASL** for fine-grained permission checks before calling Prisma.

---

## 11. Scaling MongoDB with Sharding
**Scenario:** Your MongoDB database grows rapidly. How would sharding help?

### Answer:
Sharding solves three main bottlenecks:
1. **Storage**: Splits data across multiple machines.
2. **Write Throughput**: Distributes writes across multiple primaries.
3. **Query Performance**: Parallel execution on many shards.

**Concretely:**
- **Shard Key**: Choose a high-cardinality key (e.g., `userId` or `tenantId`). Avoid monotonically increasing keys like default `ObjectId`.
- **Zone Sharding**: Can be used for data locality or compliance.
- **Balancer**: Automatically moves data chunks to maintain balance.

---

## 12. Relational Integrity vs JSON Flexibility
**Scenario:** You need both relational integrity and flexible JSON storage. Which database would you choose?

### Answer:
**PostgreSQL** is the best choice (2025–2026).

**Why:**
- Full ACID compliance + foreign keys.
- **JSONB support**: GIN indexing, containment operators, and JSONPath.
- Generated columns can be created from JSONB fields.
- CHECK constraints can enforce partial schemas on JSONB data.

It gives you relational safety where you need it and schema flexibility where requirements are fluid.

---

## 13. Debugging Production Deadlocks
**Scenario:** A production database experiences deadlocks. How would you debug and fix them?

### Answer:
**Debug Steps:**
1. Enable logs for `lock_waits` and `deadlock_timeout`.
2. Inspect `pg_locks` and `pg_stat_activity` to find blocking chains.
3. Check the "Deadlock detected" message in logs for involved queries.

**Fix Approaches:**
- **Consistent Ordering**: Always access tables/rows in the same order.
- **Shorten Transactions**: Move logic outside the DB transaction.
- **Lower Isolation**: Use `Read Committed` if `Repeatable Read` isn't strictly necessary.
- **Retry Logic**: Implement exponential backoff in the application.

---

## 14. Solving N+1 Query Problems
**Scenario:** You want to avoid the N+1 query problem. How would you handle it across Prisma and Mongoose?

### Answer:
**Prisma:**
- Use `include` or `select` for eager loading nested relations.
- Use `$transaction` or batching for multiple independent queries.
- Use **Prisma Extensions** for automated batching.

**Mongoose:**
- Use `.populate()` for related documents.
- Use `.lean()` for faster, read-only queries.
- Use the **DataLoader** pattern to batch and cache requests.

---

## 15. Redesigning Monolithic Databases
**Scenario:** You are asked to redesign a monolithic database for scalability. What factors would you consider?

### Answer:
Key factors:
- **Access Patterns**: Is it read-heavy (replicas) or write-heavy (sharding)?
- **Consistency**: Can we move to Eventual Consistency for some modules?
- **Data Volume**: Do we need Time-Series (ClickHouse) for logs/metrics?
- **Operational Overhead**: Can the team manage a distributed DB?

**Modern Path:**
- Vertical scaling first, then **Read Replicas**.
- **Polyglot Persistence**: Money in Postgres, logs in ClickHouse, cache in Redis.
- **Sharding** (Vitess/Citus) only when strictly necessary.