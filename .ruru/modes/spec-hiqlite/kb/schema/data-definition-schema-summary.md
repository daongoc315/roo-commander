# Hiqlite Data Definition & Schema

Since `hiqlite` uses SQLite as its underlying storage engine via `rqlite`, data definition and schema management are handled using standard SQLite SQL syntax. Operations are typically executed using the `execute` or `execute_batch` methods of the `HiqliteClient`.

**1. Defining Data Structures (Tables):**

*   **`CREATE TABLE` Statement:** Tables are defined using the standard SQL `CREATE TABLE` statement. This statement specifies the table name, column names, data types for each column, and any constraints (e.g., `PRIMARY KEY`, `NOT NULL`, `UNIQUE`).

    **Example:**
    ```rust
    // Assuming 'client' is an initialized HiqliteClient instance
    let _ = client.execute(
        "CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT, email TEXT UNIQUE)",
        &[] // No parameters for this DDL statement
    ).await?;
    ```
    This example creates a table named `users` if it doesn't already exist, with an integer primary key `id`, a text column `name`, and a unique text column `email`.

**2. Specifying Data Types:**

*   SQLite has a flexible, dynamic typing system. While you declare types like `INTEGER`, `TEXT`, `REAL`, `BLOB`, `NUMERIC`, SQLite will often store data as provided and convert it when necessary. The common types used in `CREATE TABLE` statements are:
    *   `TEXT`: For strings.
    *   `INTEGER`: For whole numbers.
    *   `REAL`: For floating-point numbers.
    *   `BLOB`: For binary data.
    *   `NUMERIC`: Can store values of any type.

**3. Creating and Managing Indexes:**

*   **Purpose:** Indexes are crucial for optimizing query performance, especially for `SELECT` statements with `WHERE` clauses or `ORDER BY` clauses on non-primary key columns.
*   **`CREATE INDEX` Statement:** Indexes are created using the SQL `CREATE INDEX` statement. You specify the index name, the table it applies to, and the column(s) to be indexed.

    **Example:**
    ```rust
    // Assuming 'client' is an initialized HiqliteClient instance
    // and the 'users' table exists
    let _ = client.execute(
        "CREATE INDEX IF NOT EXISTS idx_users_email ON users (email)",
        &[]
    ).await?;
    ```
    This creates an index named `idx_users_email` on the `email` column of the `users` table if the index doesn't already exist.

*   **Types of Indexes:** The provided context primarily mentions standard index creation without detailing specific SQLite index types (e.g., B-tree, which is the default for most SQLite indexes). The focus is on the `CREATE INDEX` syntax.

**4. Schema Management:**

*   Schema modifications (e.g., adding columns, altering tables) are generally done using the `ALTER TABLE` SQL command, though SQLite has some limitations on `ALTER TABLE` compared to other SQL databases.
*   Dropping tables is done with `DROP TABLE table_name;`.
*   Dropping indexes is done with `DROP INDEX index_name;`.

All these DDL (Data Definition Language) operations are sent to the `rqlite` cluster via the `hiqlite` client's execution methods. `rqlite` ensures these schema changes are replicated across the cluster via the Raft consensus mechanism.