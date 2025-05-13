# Hiqlite Advanced Data Querying

`hiqlite`, by leveraging SQLite's SQL capabilities, supports various advanced data querying patterns beyond simple row retrieval. These are achieved by constructing appropriate SQL `SELECT` statements and executing them using `hiqlite` client methods like `query_map`, `query_map_one`, or `query_map_opt`.

**1. Filtering Data (`WHERE` clause):**

*   The `WHERE` clause is used to filter rows based on specified conditions.
*   Parameters can be passed to the query to make filtering dynamic.

    **Example:**
    ```rust
    // Assuming 'client' and User struct are defined
    let min_id = 10;
    let users: Vec<User> = client.query_map(
        "SELECT id, name, email FROM users WHERE id > ?",
        &[min_id.into()]
    ).await?;
    ```

**2. Sorting Data (`ORDER BY` clause):**

*   The `ORDER BY` clause sorts the result set based on one or more columns, in ascending (`ASC`, default) or descending (`DESC`) order.

    **Example:**
    ```rust
    // Assuming 'client' and User struct are defined
    let users: Vec<User> = client.query_map(
        "SELECT id, name, email FROM users ORDER BY name DESC",
        &[]
    ).await?;
    ```

**3. Projections (Selecting Specific Fields):**

*   This is fundamental to `SELECT` statements. You specify the exact columns you want to retrieve, which can optimize data transfer and processing.

    **Example:**
    ```rust
    #[derive(serde::Deserialize, Debug)]
    struct UserName { name: String; }

    // Assuming 'client' is defined
    let names: Vec<UserName> = client.query_map(
        "SELECT name FROM users WHERE email LIKE ?",
        &["%example.com".into()]
    ).await?;
    ```

**4. Aggregations (`GROUP BY`, `COUNT`, `SUM`, `AVG`, etc.):**

*   SQL aggregate functions perform calculations on a set of rows and return a single summary row.
*   `GROUP BY` is used to group rows with the same values in specified columns, allowing aggregate functions to be applied to each group.

    **Example (Counting users):**
    ```rust
    #[derive(serde::Deserialize, Debug)]
    struct CountResult { count: i64; }

    // Assuming 'client' is defined
    let result: Option<CountResult> = client.query_map_opt(
        "SELECT COUNT(*) as count FROM users",
        &[]
    ).await?;
    if let Some(r) = result {
        println!("Total users: {}", r.count);
    }
    ```

**5. Joining Related Data (`JOIN` clauses):**

*   SQLite supports various `JOIN` clauses (`INNER JOIN`, `LEFT JOIN`, etc.) to combine rows from two or more tables based on related columns.
*   While the provided context files for `hiqlite` don't explicitly show complex `JOIN` examples, standard SQLite `JOIN` syntax would be used in the SQL query string passed to `hiqlite` methods.

    **Conceptual Example (assuming an `orders` table):**
    ```sql
    -- This SQL would be passed to a hiqlite query method
    SELECT users.name, orders.item
    FROM users
    INNER JOIN orders ON users.id = orders.user_id
    WHERE users.id = ?;
    ```
    You would then define a Rust struct to deserialize the joined result.

**6. Limiting Results (`LIMIT` clause):**

*   The `LIMIT` clause restricts the number of rows returned by a query. Often used with `OFFSET` for pagination.

    **Example:**
    ```rust
    // Assuming 'client' and User struct are defined
    let users: Vec<User> = client.query_map(
        "SELECT id, name, email FROM users ORDER BY id LIMIT 10 OFFSET 20", // Get 10 users, skipping the first 20
        &[]
    ).await?;
    ```

These patterns are standard SQL features available through `hiqlite` because it interfaces with `rqlite`, which uses SQLite. The `hiqlite` client provides the means to execute these SQL statements and map their results to Rust types.