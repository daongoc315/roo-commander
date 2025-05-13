# Hiqlite Backup & Recovery

The provided context documents for `hiqlite` (the Rust client for `rqlite`) offer limited direct information on specific backup and recovery procedures. However, some inferences can be made based on its architecture and underlying technologies (`rqlite`, SQLite, Raft).

**Key Points from Provided Context:**

*   **`last_applied_log_id` Persistence:** The `hiqlite` (referring to the underlying system it's built upon, likely `rqlite` or a similar Raft-based SQLite system) documentation mentions that the `last_applied_log_id` from Raft messages is persisted to the database during **snapshot creation or backups**. This implies that backup mechanisms are tied into the Raft consensus and state machine snapshotting process.
*   **Auto-Heal Feature:** Hiqlite (the system) has an "auto-heal feature" for scenarios like ungraceful shutdowns, which suggests some level of built-in resilience and potential for state recovery.

**Inferred Backup and Recovery Considerations (General for Raft-based Systems like `rqlite`):**

While not explicitly detailed for `hiqlite` client usage in the provided snippets, systems like `rqlite` typically support backup and recovery through:

1.  **Snapshots:**
    *   Raft-based systems periodically take snapshots of the state machine (in this case, the SQLite database) to compact the Raft log. These snapshots are a crucial part of the recovery process for nodes rejoining a cluster or for restoring the system.
    *   Backups often involve securely storing these snapshots.

2.  **`rqlite` Specifics (External Knowledge - `rqlite` documentation would confirm):**
    *   `rqlite` itself provides API endpoints for backups (e.g., `/db/backup`). These typically stream a SQLite backup file.
    *   Restoration usually involves stopping the `rqlite` node (or cluster), replacing the SQLite database file with a backup, and restarting the node/cluster, allowing Raft to re-establish consistency if part of a cluster.

**`hiqlite` Client Role:**
*   The `hiqlite` client itself is primarily for application-level interaction (CRUD, queries). It is unlikely to have direct API calls for initiating full database backups or restores of the `rqlite` cluster. These are typically administrative operations performed directly against the `rqlite` server nodes or cluster management interface.

**Summary from Provided Context:**
*   The `hiqlite` system (likely referring to the server-side component it interacts with) incorporates backup mechanisms that are linked to its Raft log and snapshotting process.
*   An auto-heal feature exists, suggesting some fault tolerance and recovery capabilities.

For detailed backup and recovery procedures, one would need to consult the documentation for `rqlite` (the distributed database system that `hiqlite` client connects to) or the specific Raft-based SQLite implementation being used, rather than just the `hiqlite` client library documentation. The provided snippets do not offer instructions on how to trigger or manage backups/restores using the `hiqlite` Rust client.