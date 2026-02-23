# MySQL / PostgreSQL Questions

## 1. Explain the key architectural differences between PostgreSQL and MySQL.
PostgreSQL uses a process-based architecture where each connection spawns a new process, enabling robust concurrency and extensibility, while MySQL employs a thread-based model for lighter resource usage and faster performance in read-heavy workloads. PostgreSQL emphasizes standards compliance and advanced features like full-text search natively, whereas MySQL focuses on simplicity and speed, with plugins for extensibility. In practice, PostgreSQL suits complex, transactional apps, and MySQL excels in web-scale, high-availability scenarios.

## 2. What are primary keys and foreign keys?
A primary key is a unique identifier for each record in a table, ensuring no duplicates and enabling fast lookups—often an auto-incrementing ID. A foreign key links two tables by referencing a primary key in another, enforcing referential integrity to prevent orphaned records. This maintains data consistency, like linking orders to customers in an e-commerce database, reducing errors and supporting relational queries.

## 3. How do transactions work in relational databases?
Transactions group multiple SQL operations into a single atomic unit using ACID properties: Atomicity ensures all or none succeed, Consistency maintains database rules, Isolation prevents interference, and Durability persists changes post-commit. You start with BEGIN, execute statements, then COMMIT or ROLLBACK. This is crucial for business operations like bank transfers, where partial failures could lead to inconsistencies.

## 4. Explain isolation levels and their impact.
Isolation levels control how transactions see each other's changes, balancing consistency and performance. READ UNCOMMITTED allows dirty reads for max concurrency but risks inconsistencies; READ COMMITTED prevents dirty reads but allows non-repeatable reads; REPEATABLE READ avoids that but permits phantoms; SERIALIZABLE ensures full isolation at the cost of performance. Higher levels reduce anomalies but increase locking, impacting throughput in high-traffic apps.

## 5. What is MVCC, and how does it help concurrency?
Multiversion Concurrency Control (MVCC) maintains multiple versions of data rows, allowing readers to access consistent snapshots without blocking writers. PostgreSQL uses this natively, reducing locks and enabling higher concurrency compared to MySQL's default locking. It helps in busy systems like analytics platforms by minimizing wait times, though it requires vacuuming to manage old versions and prevent bloat.

## 6. Explain different types of joins with use cases.
INNER JOIN returns matching rows from both tables, useful for combining user profiles with orders. LEFT JOIN includes all left table rows plus matches, ideal for listing all products with optional sales data. RIGHT JOIN mirrors LEFT but for the right table. FULL OUTER JOIN combines both, helpful for merging datasets with gaps. CROSS JOIN produces Cartesian products for combinatorial analysis, like generating all possible pairings.

## 7. How does indexing improve query performance?
Indexing creates a data structure (like B-trees) for quick lookups, reducing full table scans to logarithmic time complexity. It speeds up WHERE, JOIN, and ORDER BY clauses, but adds overhead for inserts/updates. In a large user database, indexing email columns accelerates logins, balancing query speed with write performance for overall efficiency.

## 8. What are composite indexes, and when should they be used?
Composite indexes cover multiple columns in one index, optimizing queries filtering or sorting on those combinations. Use them when queries frequently use the same column set in WHERE clauses, like last_name and first_name for user searches. They enforce column order for efficiency, reducing I/O in reports or analytics, but avoid over-indexing to prevent maintenance costs.

## 9. How do PostgreSQL and MySQL differ in JSON support?
PostgreSQL offers native JSONB for binary storage with indexing and operators, enabling efficient querying and validation—great for semi-structured data like user preferences. MySQL uses JSON type with functions but less advanced indexing, suitable for simpler storage. PostgreSQL's approach supports complex apps needing NoSQL-like flexibility within SQL, while MySQL prioritizes ease.

## 10. What are views, and why are they useful?
Views are virtual tables defined by a query, abstracting complex joins or aggregations without storing data. They're useful for security (restricting access), simplifying queries for reports, and maintaining consistency across apps. For instance, a sales view hides raw tables, improving readability and reducing errors in business intelligence tools.

## 11. What are stored procedures and triggers?
Stored procedures are precompiled SQL code blocks executed as units, encapsulating logic for reuse and security, like processing orders. Triggers are automatic actions on events (insert/update/delete), enforcing rules like auditing changes. They centralize business logic, reducing app-side code and ensuring data integrity in enterprise systems.

## 12. What is normalization vs denormalization?
Normalization organizes data into tables to minimize redundancy and anomalies via forms (1NF-5NF), improving integrity but potentially slowing queries with joins. Denormalization adds redundant data for read performance, like caching aggregates. Normalize for write-heavy OLTP systems; denormalize for read-heavy OLAP to balance speed and maintenance.

## 13. How does replication work in relational databases?
Replication copies data from a primary (master) to secondary (slaves) nodes for redundancy and load balancing. MySQL uses binlog-based async replication; PostgreSQL supports streaming for sync/async. It enables high availability, backups, and scaling reads, critical for e-commerce sites to handle failures without downtime.

## 14. What causes deadlocks in SQL databases?
Deadlocks occur when transactions hold locks and wait for each other's resources in a cycle, like two processes updating swapped rows. Causes include poor query order, long transactions, or high concurrency without proper indexing. They halt progress, requiring detection and resolution, impacting user experience in multi-user apps.

## 15. How can deadlocks be prevented or minimized?
Prevent deadlocks by accessing resources in consistent order, keeping transactions short, using appropriate isolation levels, and indexing to reduce lock times. Monitor with tools like SHOW ENGINE INNODB STATUS in MySQL, and implement retry logic in apps. This maintains performance in concurrent environments like financial systems.
