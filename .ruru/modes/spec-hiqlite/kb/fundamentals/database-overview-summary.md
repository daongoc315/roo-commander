# Hiqlite Overview & Data Model

**Database Type:**
Hiqlite enhances SQLite, making it a **distributed relational database**. It builds upon SQLite's familiar SQL interface and features.

**Core Data Model:**
As it uses SQLite as its storage engine, Hiqlite's fundamental data model is based on **tables with rows and columns**, adhering to a relational schema.

**Key Features:**
*   **High Availability:** Ensures the database remains operational despite node failures in a cluster. This is a primary design goal.
*   **Embeddable:** Can be embedded directly within an application, similar to SQLite.
*   **Raft-Based Consensus:** Utilizes the Raft consensus algorithm for data replication and leader election across the distributed cluster, ensuring data consistency.
*   **SQLite Integration:** Leverages the standard SQL interface and features of SQLite.
*   **Caching:** Incorporates optional in-memory caching mechanisms for `SELECT` queries to improve read performance. The cache has configurable TTL and max size, using an LRU eviction policy.
*   **Asynchronous Rust Client:** `hiqlite` itself is a high-level, asynchronous Rust client for `rqlite` (the distributed system using SQLite). It simplifies interactions with features like connection pooling and automatic JSON serialization/deserialization.
*   **Leader Redirection & Node Discovery:** The client automatically handles leader redirection in a multi-node `rqlite` setup.
*   **Optimized I/O:** Reduces I/O for Raft log persistence by writing `last_applied_log_id` to the database primarily during snapshots or backups, with an auto-heal feature for ungraceful shutdowns.

**Typical Use Cases:**
Hiqlite is suitable for applications requiring:
*   A resilient and fault-tolerant relational data storage solution.
*   Data persistence and availability even in the event of node failures.
*   An embedded database solution with high availability characteristics.
*   Simplified interaction with a distributed `rqlite` cluster from Rust applications.