# Knowledge Base for spec-hiqlite Mode

This Knowledge Base (KB) provides information for the `spec-hiqlite` mode. This mode is an expert on `hiqlite` (https://github.com/sebadob/hiqlite), focusing on its architecture, best practices for usage within Rust projects, including creating multi-node setups and client instances.

The documents herein detail various aspects of `hiqlite` to support the mode's operations.

## KB Files

Below is a list of available documents in this knowledge base:

*   **[`client/setup-connection-summary.md`](./client/setup-connection-summary.md)** (68 lines)
    *   Summary: This document explains how to set up and connect Hiqlite, a Rust client for `rqlite`. It covers installation via `Cargo.toml`, establishing connections by creating a `HiqliteClient` instance with node addresses and `HiqliteClientConfig` (including node config, timeout, consistency level, and optional caching). An example of client initialization is provided. It also mentions Basic Authentication as the primary method and discusses considerations for multi-node `rqlite` setups, including automatic leader redirection and connection pooling.
*   **[`fundamentals/database-overview-summary.md`](./fundamentals/database-overview-summary.md)** (24 lines)
    *   Summary: This document provides an overview of Hiqlite, describing it as a distributed relational database enhancing SQLite. It details its core data model (tables with rows and columns), key features (high availability, embeddability, Raft-based consensus, SQLite integration, caching, asynchronous Rust client, leader redirection, optimized I/O), and typical use cases (resilient fault-tolerant relational data storage, data persistence with high availability, embedded database with HA, simplified interaction with distributed rqlite from Rust).
*   **[`operations/backup-recovery-summary.md`](./operations/backup-recovery-summary.md)** (29 lines)
    *   Summary: This document discusses backup and recovery for Hiqlite. It notes limited direct information in the provided context but infers procedures based on `rqlite`, SQLite, and Raft. Key points include `last_applied_log_id` persistence during snapshots/backups and an auto-heal feature. It suggests that Raft-based systems like `rqlite` use snapshots for recovery and that `rqlite` has API endpoints for backups. The `hiqlite` client itself is unlikely to manage these administrative operations directly; `rqlite` documentation should be consulted for detailed procedures.
*   **[`performance/indexing-summary.md`](./performance/indexing-summary.md)** (43 lines)
    *   Summary: This document explains the importance of indexing for query performance in `hiqlite` (which uses SQLite via `rqlite`). It details how indexes speed up `SELECT` queries by avoiding full table scans. The document shows how to create indexes using the standard SQL `CREATE INDEX` statement via `hiqlite`'s `client.execute()` method, including syntax and an example. It briefly touches on index types, noting SQLite typically uses B-trees. Performance considerations emphasize leveraging local reads and ensuring the schema includes appropriate indexes.
*   **[`schema/data-definition-schema-summary.md`](./schema/data-definition-schema-summary.md)** (52 lines)
    *   Summary: This document outlines data definition and schema management in `hiqlite`, which uses SQLite. It covers defining tables with `CREATE TABLE`, specifying SQLite data types (`TEXT`, `INTEGER`, `REAL`, `BLOB`, `NUMERIC`), creating indexes with `CREATE INDEX` for performance, and general schema management using `ALTER TABLE`, `DROP TABLE`, and `DROP INDEX`. All DDL operations are sent to the `rqlite` cluster via `hiqlite` client's execution methods and replicated via Raft.
*   **[`security/security-access-control-summary.md`](./security/security-access-control-summary.md)** (49 lines)
    *   Summary: This document covers security and access control for `hiqlite`. It details client-to-node authentication using Basic Authentication (username/password in `HiqliteNodeConfig`) and secure transport via HTTPS (configured by setting `scheme` to `Scheme::Https`). It notes that features like database-level user roles/permissions, Row-Level Security (RLS), network access controls beyond client auth/TLS, and data encryption at rest are not explicitly detailed for the `hiqlite` client and would depend on the `rqlite` server or underlying SQLite capabilities.
*   **[`transactions/transactions-summary.md`](./transactions/transactions-summary.md)** (44 lines)
    *   Summary: This document explains that `hiqlite` supports atomic operations through standard SQL transactions (`BEGIN`, `COMMIT`, `ROLLBACK`), executed via the client's `execute()` method. It provides a conceptual example flow. It also notes that `rqlite`'s Raft consensus ensures these transactional changes are consistently replicated. The document mentions `hiqlite`'s batching for performance and the role of `consistency_level` in client configuration.
*   **[`usage/basic-crud-summary.md`](./usage/basic-crud-summary.md)** (102 lines)
    *   Summary: This document details basic CRUD (Create, Read, Update, Delete) operations in `hiqlite`. It lists core client methods like `execute()` for modifications/DDL, and `query_map()`, `query_map_one()`, `query_map_opt()` for `SELECT` queries, mapping results to Rust structs. Examples are provided for `INSERT`, `SELECT` (multiple and single/optional results), `UPDATE`, and `DELETE` statements, showing how to pass parameters and handle results.
*   **[`usage/querying-data-summary.md`](./usage/querying-data-summary.md)** (97 lines)
    *   Summary: This document describes advanced data querying in `hiqlite` using SQLite's SQL capabilities. It covers filtering with `WHERE`, sorting with `ORDER BY`, projections (selecting specific fields), aggregations (`GROUP BY`, `COUNT`, etc.), joining data with `JOIN` clauses (conceptual example), and limiting results with `LIMIT` (often with `OFFSET` for pagination). Examples are provided for each pattern, demonstrating execution via `hiqlite` client methods.

### Examples (v0.6.0)

These documents provide practical code examples based on Hiqlite version 0.6.0.

*   **[`examples/sql-only-v0.6.0.md`](./examples/sql-only-v0.6.0.md)** (231 lines)
    *   Summary: Provides examples for using Hiqlite in a project primarily focusing on its SQLite capabilities, including setup, migrations, basic operations (execute, insert, query), transactions (simple and with `RETURNING`/`StmtIndex`), and batch execution.
*   **[`examples/cache-usage-v0.6.0.md`](./examples/cache-usage-v0.6.0.md)** (133 lines)
    *   Summary: Demonstrates usage of Hiqlite's in-memory caching features, including setup (dependencies, CacheEnum, CacheIndex), client initialization with cache, and cache operations like put, get, delete, clear specific cache, and clear all caches.
*   **[`examples/full-features-v0.6.0.md`](./examples/full-features-v0.6.0.md)** (150 lines)
    *   Summary: Showcases combined usage of Hiqlite's features: SQLite database operations, in-memory caching, distributed locks (`dlock`), and listen/notify capabilities, based on the `examples/walkthrough/src/main.rs` example.