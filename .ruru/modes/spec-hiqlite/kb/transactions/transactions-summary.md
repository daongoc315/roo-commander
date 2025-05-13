# Hiqlite Transactions & Atomicity

`hiqlite`, interacting with `rqlite` which uses SQLite as its engine, supports atomic operations through SQL transactions. This ensures that a series of database changes are treated as a single unit: either all changes are successfully applied (committed), or none are (rolled back), maintaining data integrity.

**Mechanism for Atomicity:**

*   **SQL Transactions:** Atomicity is achieved using standard SQL transaction control statements.
    *   **`BEGIN TRANSACTION` (or simply `BEGIN`):** Starts a new transaction. Subsequent database operations are part of this transaction.
    *   **`COMMIT`:** Finalizes the transaction, making all changes permanent.
    *   **`ROLLBACK`:** Aborts the transaction, undoing any changes made since `BEGIN`.

*   **Execution via `hiqlite`:** These SQL transaction control statements are sent to the `rqlite` cluster using the `hiqlite` client's `execute()` method.

**Example Flow (Conceptual):**

While the provided `hiqlite` documents don't show a complete transaction block example using `BEGIN/COMMIT/ROLLBACK` directly within Rust code snippets, the underlying SQLite engine fully supports them. A typical sequence executed via `hiqlite` would be:

1.  Execute `BEGIN TRANSACTION;`
    ```rust
    // client.execute("BEGIN TRANSACTION", &[]).await?;
    ```
2.  Execute one or more data modification statements (`INSERT`, `UPDATE`, `DELETE`).
    ```rust
    // client.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 1", &[]).await?;
    // client.execute("UPDATE accounts SET balance = balance + 100 WHERE id = 2", &[]).await?;
    ```
3.  Based on the success of these operations and application logic:
    *   Execute `COMMIT;` to make changes permanent.
        ```rust
        // client.execute("COMMIT", &[]).await?;
        ```
    *   Or, execute `ROLLBACK;` to undo changes if an error occurred or a condition was not met.
        ```rust
        // client.execute("ROLLBACK", &[]).await?;
        ```

**Raft Consensus and Transactions:**
*   `rqlite` uses the Raft consensus algorithm. When transaction statements like `COMMIT` are processed by the leader, Raft ensures that these changes (or the decision to commit/rollback) are consistently replicated across the cluster before being applied, thus maintaining atomicity even in a distributed environment.

**Hiqlite Specifics:**
*   The `hiqlite` documentation mentions batching writes for performance. While not explicitly detailed as "transactions" in the snippets, batching operations sent via `execute_batch` would typically be wrapped in a transaction by `rqlite` itself or should be by the user for atomicity if multiple distinct SQL statements are in the batch.
*   The `consistency_level` setting in `HiqliteClientConfig` (e.g., "strong") ensures that operations, including those within transactions, are handled with the appropriate level of consistency across the `rqlite` cluster.

In essence, `hiqlite` users rely on standard SQLite transaction commands, executed through the client, to ensure atomicity for operations involving multiple data changes. `rqlite` then ensures these transactional changes are consistently managed across its distributed nodes.