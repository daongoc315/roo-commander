# Hiqlite Indexing for Performance

Indexes are critical database structures for optimizing query performance in systems using `hiqlite` (and its underlying SQLite engine via `rqlite`). They allow the database to find rows matching query conditions more quickly, avoiding full table scans.

**Importance of Indexes:**

*   **Faster Queries:** Indexes significantly speed up `SELECT` queries, especially those with `WHERE` clauses, `JOIN` conditions, or `ORDER BY` clauses on indexed columns.
*   **Efficient Data Retrieval:** By creating a sorted data structure (or a structure allowing quick lookups) for specific columns, the database can locate relevant data with fewer I/O operations.

**Creating Indexes:**

*   Indexes are created using the standard SQL `CREATE INDEX` statement. This DDL command is executed via `hiqlite`'s `client.execute()` method.

    **Syntax:**
    ```sql
    CREATE INDEX IF NOT EXISTS index_name ON table_name (column1, column2, ...);
    ```

    *   `IF NOT EXISTS`: Optional clause to prevent an error if the index already exists.
    *   `index_name`: A unique name for the index.
    *   `table_name`: The table on which the index is created.
    *   `(column1, column2, ...)`: The column(s) to include in the index.

    **Example (from provided context):**
    ```rust
    // Assuming 'client' is an initialized HiqliteClient instance
    let _ = client.execute(
        "CREATE INDEX IF NOT EXISTS idx_users_email ON users (email)",
        &[]
    ).await?;
    ```
    This creates an index named `idx_users_email` on the `email` column of the `users` table.

**Types of Indexes:**

*   The provided context primarily demonstrates the standard `CREATE INDEX` syntax. While SQLite supports various index types internally (B-tree is common), the `hiqlite` documentation snippets do not delve into specifying different index types (e.g., Hash, GIN, GiST, which are more common in other database systems like PostgreSQL). For `hiqlite` interacting with SQLite, one would typically rely on SQLite's default indexing mechanisms (usually B-tree) unless specific SQLite PRAGMAs or advanced syntax (not shown in context) are used.

**Performance Considerations with Hiqlite:**

*   **Leverage Local Reads:** Hiqlite aims for fast local reads. Well-chosen indexes are crucial to ensure these local reads are efficient.
*   **SQLite Configuration:** The underlying SQLite performance is key. While `hiqlite` abstracts direct SQLite interaction, ensuring the database schema includes appropriate indexes is a primary way to influence this performance.

In summary, for `hiqlite`, indexing is managed through standard SQL `CREATE INDEX` commands sent to the `rqlite` cluster. These indexes are vital for achieving good query performance by enabling faster data retrieval from the SQLite engine.