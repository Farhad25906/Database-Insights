# Prisma Questions

## 📜 Table of Contents
- [1. What is Prisma, and how does it differ from traditional ORMs?](#1-what-is-prisma-and-how-does-it-differ-from-traditional-orms)
- [2. Explain the Prisma schema file and its purpose.](#2-explain-the-prisma-schema-file-and-its-purpose)
- [3. How do Prisma Client provide type safety?](#3-how-do-prisma-client-provide-type-safety)
- [4. How are models and relations defined in Prisma?](#4-how-are-models-and-relations-defined-in-prisma)
- [5. What are Prisma enums, and why are they useful?](#5-what-are-prisma-enums-and-why-are-they-useful)
- [6. How does Prisma handle CRUD operations?](#6-how-does-prisma-handle-crud-operations)
- [7. Explain nested reads and nested writes in Prisma.](#7-explain-nested-reads-and-nested-writes-in-prisma)
- [8. How does Prisma manage database transactions?](#8-how-does-prisma-manage-database-transactions)
- [9. What happens when a Prisma transaction fails?](#9-what-happens-when-a-prisma-transaction-fails)
- [10. What is Prisma Migrate?](#10-what-is-prisma-migrate)
- [11. Difference between migrate dev and db push.](#11-difference-between-migrate-dev-and-db-push)
- [12. How does Prisma prevent SQL injection?](#12-how-does-prisma-prevent-sql-injection)
- [13. How does Prisma handle pagination and filtering?](#13-how-does-prisma-handle-pagination-and-filtering)
- [14. How does Prisma work with MySQL vs PostgreSQL?](#14-how-does-prisma-work-with-mysql-vs-postgresql)
- [15. What are common limitations or pitfalls of Prisma?](#15-what-are-common-limitations-or-pitfalls-of-prisma)

## 1. What is Prisma, and how does it differ from traditional ORMs?
Prisma is a modern ORM for Node.js/TypeScript that auto-generates a type-safe client from a schema, focusing on developer experience. Unlike traditional ORMs like Sequelize, which use runtime queries, Prisma uses a declarative schema and compile-time checks, reducing errors and boilerplate while integrating seamlessly with databases for efficient, business-aligned development.

## 2. Explain the Prisma schema file and its purpose.
The Prisma schema (.prisma file) defines data models, relations, enums, and datasource/providers declaratively. It generates migrations and the Prisma Client, ensuring consistency between code and database. This centralizes schema management, streamlining development for teams building scalable apps like SaaS platforms.

## 3. How does Prisma Client provide type safety?
Prisma Client generates TypeScript types from the schema, enabling autocompletion and compile-time error checking for queries. This prevents runtime issues like invalid fields, fostering reliable code in full-stack projects where data integrity directly impacts user trust and business outcomes.

## 4. How are models and relations defined in Prisma?
Models are defined as blocks with fields (e.g., id Int @id), and relations use references like user User @relation(fields: [userId], references: [id]). This handles one-to-many, many-to-many via implicit tables, simplifying relational data modeling for applications like social networks.

## 5. What are Prisma enums, and why are they useful?
Prisma enums define fixed value sets in the schema, like enum Role { ADMIN USER }, mapping to database enums or checks. They're useful for type-safe constraints, reducing invalid data and improving code readability in systems requiring role-based access control.

## 6. How does Prisma handle CRUD operations?
Prisma provides intuitive methods like create, findMany, update, delete on the client, with options for filtering and relations. It abstracts SQL, ensuring efficient execution while allowing business logic focus, like managing inventory in an e-commerce backend.

## 7. Explain nested reads and nested writes in Prisma.
Nested reads use include/select for eager loading relations, like finding users with posts. Nested writes handle creates/updates across relations atomically, such as creating a post with author. This simplifies complex operations, enhancing performance in data-intensive apps.

## 8. How does Prisma manage database transactions?
Prisma supports interactive transactions via $transaction, executing multiple operations atomically. It ensures consistency for sequences like debit-credit, integrating with business processes to prevent partial failures in transactional systems.

## 9. What happens when a Prisma transaction fails?
On failure, Prisma rolls back all operations within the transaction, throwing an error for handling. This maintains data integrity, allowing retries or logging, crucial for reliable service in high-stakes environments like finance.

## 10. What is Prisma Migrate?
Prisma Migrate is a tool for schema migrations, generating SQL from schema changes via commands like migrate dev. It handles versioning and deployments, ensuring smooth evolution of databases in agile teams.

## 11. Difference between migrate dev and db push.
Migrate dev creates migration files for review and applies them, ideal for development with history. Db push directly syncs schema to database without files, faster for prototyping but less controlled for production, balancing speed and safety.

## 12. How does Prisma prevent SQL injection?
Prisma uses parameterized queries and escapes inputs automatically, eliminating manual sanitization. This built-in security protects against attacks, ensuring compliance and trust in applications handling sensitive data.

## 13. How does Prisma handle pagination and filtering?
Prisma supports skip/take for pagination and where clauses for filtering, with orderBy for sorting. It optimizes queries server-side, efficient for large datasets in user-facing apps like search features.

## 14. How does Prisma work with MySQL vs PostgreSQL?
Prisma abstracts differences via providers in the schema, supporting features like JSON in both but leveraging PostgreSQL's advanced types (e.g., enums natively). It ensures cross-database compatibility, allowing flexibility in choosing based on workload needs.

## 15. What are common limitations or pitfalls of Prisma?
Common pitfalls include over-nesting queries causing performance issues, schema drift if not migrated properly, and limited raw SQL support for complex cases. It may add overhead in very high-throughput scenarios, so monitor queries and use batching to maintain efficiency in production.
