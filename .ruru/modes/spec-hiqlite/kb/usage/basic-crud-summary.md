# Hiqlite Basic CRUD Operations

`hiqlite` facilitates Creating, Reading, Updating, and Deleting (CRUD) data in an `rqlite` (and thus SQLite) database using SQL statements executed via its client methods.

**Core Client Methods for CRUD:**

*   **`client.execute(sql: &str, params: &[Value]) -> Result<ExecResults>`:**
    Used for SQL statements that modify data (`INSERT`, `UPDATE`, `DELETE`) or DDL statements. Returns `ExecResults` which includes `last_insert_id` and `rows_affected`.
*   **`client.query_map<T: DeserializeOwned + Send>(sql: &str, params: &[Value]) -> Result<Vec<T>>`:**
    Executes a `SELECT` query and maps each row to a struct `T` that implements `serde::Deserialize`.
*   **`client.query_map_one<T: DeserializeOwned + Send>(sql: &str, params: &[Value]) -> Result<T>`:**
    Similar to `query_map`, but expects exactly one result row.
*   **`client.query_map_opt<T: DeserializeOwned + Send>(sql: &str, params: &[Value]) -> Result<Option<T>>`:**
    Similar to `query_map`, but expects zero or one result row.
*   **`client.query_json<T: DeserializeOwned + Send>(sql: &str, params: &[Value]) -> Result<Vec<T>>`:**
    (Mentioned in cache context, implies similar functionality to `query_map` but might handle JSON types more directly or be an older/alternative naming).
*   **Parameters (`params: &[Value]`):** SQL parameters are passed as a slice of `hiqlite::Value`, which can be created from common Rust types using `.into()`.

**1. Create (Insert Data):**

*   Uses the SQL `INSERT INTO` statement.
*   Executed via `client.execute()`.

    **Example:**
    ```rust
    // Assuming 'client' is an initialized HiqliteClient instance
    let name = "Alice";
    let email = "alice@example.com";
    let results = client.execute(
        "INSERT INTO users (name, email) VALUES (?, ?)",
        &[name.into(), email.into()]
    ).await?;
    println!("Inserted ID: {:?}, Rows affected: {}", results.last_insert_id, results.rows_affected);
    ```

**2. Read (Query Data):**

*   Uses the SQL `SELECT` statement.
*   Executed via `query_map`, `query_map_one`, or `query_map_opt` for mapping to Rust structs.

    **Example (Fetching multiple users):**
    ```rust
    use serde::Deserialize;

    #[derive(Deserialize, Debug)]
    struct User {
        id: i64,
        name: String,
        email: String,
    }

    // Assuming 'client' is an initialized HiqliteClient instance
    let users: Vec<User> = client.query_map("SELECT id, name, email FROM users", &[]).await?;
    for user in users {
        println!("{:?}", user);
    }
    ```

    **Example (Fetching a single user by ID):**
    ```rust
    // Assuming 'client' and User struct from above
    let user_id = 1;
    let user: Option<User> = client.query_map_opt("SELECT id, name, email FROM users WHERE id = ?", &[user_id.into()]).await?;
    match user {
        Some(u) => println!("Found user: {:?}", u),
        None => println!("User with ID {} not found", user_id),
    }
    ```

**3. Update (Modify Data):**

*   Uses the SQL `UPDATE` statement.
*   Executed via `client.execute()`.

    **Example:**
    ```rust
    // Assuming 'client' is an initialized HiqliteClient instance
    let new_email = "alice_new@example.com";
    let user_id = 1;
    let results = client.execute(
        "UPDATE users SET email = ? WHERE id = ?",
        &[new_email.into(), user_id.into()]
    ).await?;
    println!("Rows updated: {}", results.rows_affected);
    ```

**4. Delete (Remove Data):**

*   Uses the SQL `DELETE FROM` statement.
*   Executed via `client.execute()`.

    **Example:**
    ```rust
    // Assuming 'client' is an initialized HiqliteClient instance
    let user_id = 1;
    let results = client.execute(
        "DELETE FROM users WHERE id = ?",
        &[user_id.into()]
    ).await?;
    println!("Rows deleted: {}", results.rows_affected);
    ```
These examples demonstrate the fundamental CRUD operations using `hiqlite` by constructing appropriate SQL queries and using the client's methods for execution and result mapping.