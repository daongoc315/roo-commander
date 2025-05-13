# Hiqlite: SQL-Only Project Examples (v0.6.0)

This document provides examples for using Hiqlite in a project that primarily focuses on its SQLite capabilities, without necessarily leveraging cache, distributed locks, or listen/notify features. These examples are based on `v0.6.0` and largely derived from `examples/sqlite-only/src/main.rs`.

## 1. Setup

### Dependencies

Ensure your `Cargo.toml` includes `hiqlite` and `hiqlite-macros`. `serde` is useful for custom structs, and `tokio` for the async runtime.

```toml
[dependencies]
hiqlite = { version = "0.6.0", features = ["shutdown-handle"] } # Or path to local version
hiqlite-macros = { version = "0.6.0" } # Or path to local version

serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1", features = ["rt-multi-thread"] }
# tracing and tracing-subscriber for logging (optional)
```

### Migrations

Define your SQL migrations in a directory (e.g., `migrations/`) and embed them:

```rust
use hiqlite_macros::embed::*;

#[derive(Embed)]
#[folder = "migrations"]
struct Migrations;
```
*Example from [`examples/sqlite-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:999) (repomix output).*

### Entity Struct

Define a struct for your data, optionally deriving `serde::Serialize`, `serde::Deserialize`, and implementing `From<hiqlite::Row>` for easy mapping.

```rust
use hiqlite::Row;
use serde::{Deserialize, Serialize};
use std::fmt::Debug;

#[derive(Debug, Serialize, Deserialize)]
struct Entity {
    pub id: String,
    pub num: i64,
    pub description: Option<String>,
}

impl<'r> From<Row<'r>> for Entity {
    fn from(mut row: Row<'r>) -> Self {
        Self {
            id: row.get("id"),
            num: row.get("num"),
            description: row.get("description"),
        }
    }
}
```
*Example from [`examples/sqlite-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1002) (repomix output).*

## 2. Client Initialization and Basic Operations

```rust
use hiqlite::{Client, Error, NodeConfig, Params, StmtIndex}; // StmtIndex for advanced transactions
use hiqlite_macros::params; // For the params! macro
// Assuming Migrations and Entity structs are defined as above

#[tokio::main]
async fn main() -> Result<(), Error> {
    // Initialize logging (optional)
    // tracing_subscriber::fmt().with_env_filter(EnvFilter::from("info")).init();

    // Load NodeConfig (e.g., from a file or created programmatically)
    let config = NodeConfig::from_env_file("config"); // Assumes a 'config' file
    
    // Start the Hiqlite node and get a client
    let client = hiqlite::start_node(config).await?;

    // Register a shutdown handle for graceful cleanup
    let mut shutdown_handle = client.shutdown_handle()?;

    println!("Applying database migrations...");
    client.migrate::<Migrations>().await?;

    println!("Ensuring table is empty from previous runs...");
    client.execute("DELETE FROM test", params!()).await?;

    // ... further operations ...

    println!("Operations complete. Waiting for shutdown signal (e.g., CTRL+C)...");
    shutdown_handle.wait().await?;
    Ok(())
}
```
*Source: [`examples/sqlite-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1017) (repomix output).*

## 3. Executing Statements

### Simple Execute (No Rows Returned)

For `INSERT`, `UPDATE`, `DELETE` statements that don't return data.

```rust
// async fn insert_example(client: &Client) -> Result<usize, Error> {
    let rows_affected = client
        .execute(
            "INSERT INTO test (id, num, description) VALUES ($1, $2, $3)",
            params!("id1", 123, "my description for 1. row"),
        )
        .await?;
    println!("Inserted {} row(s)", rows_affected);
//     Ok(rows_affected)
// }
```
*Source: [`examples/sqlite-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1032) (repomix output).*

### Execute with `RETURNING`

If your `INSERT` or `UPDATE` statement has a `RETURNING` clause, use `execute_returning_map_one` (for a single expected row) or `execute_returning_map` (for multiple rows).

```rust
// Assuming an 'items' table with 'id INTEGER PRIMARY KEY, name TEXT'
// async fn insert_and_return_item(client: &Client, item_name: &str) -> Result<i64, Error> {
//     #[derive(Debug)]
//     struct ItemId { id: i64 }
//     impl<'r> From<hiqlite::Row<'r>> for ItemId {
//         fn from(mut row: hiqlite::Row<'r>) -> Self { Self { id: row.get("id") } }
//     }
//
//     let returned_item_id: ItemId = client.execute_returning_map_one(
//         "INSERT INTO items (name) VALUES ($1) RETURNING id",
//         params!(item_name),
//     ).await?;
//     println!("Inserted item with ID: {}", returned_item_id.id);
//     Ok(returned_item_id.id)
// }
```
*Conceptual example based on `execute_returning_map_one` from [`hiqlite/src/client/execute.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:2456).*

## 4. Querying Data

### Query Single Row, Map to Struct (`query_as_one`)

Deserializes directly into a struct that implements `serde::Deserialize`.

```rust
// async fn get_entity_as(client: &Client, entity_id: &str) -> Result<Entity, Error> {
    let entity: Entity = client
        .query_as_one("SELECT * FROM test WHERE id = $1", params!(entity_id))
        .await?;
    // println!("Fetched (query_as_one): {:?}", entity);
//     Ok(entity)
// }
```
*Source: [`examples/sqlite-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1040) (repomix output).*

### Query Single Row, Map with `From<Row>` (`query_map_one`)

Uses the `From<hiqlite::Row>` trait implementation for mapping.

```rust
// async fn get_entity_map(client: &Client, entity_id: &str) -> Result<Entity, Error> {
    let entity: Entity = client
        .query_map_one("SELECT * FROM test WHERE id = $1", params!(entity_id))
        .await?;
    // println!("Fetched (query_map_one): {:?}", entity);
//     Ok(entity)
// }
```
*Source: [`examples/sqlite-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1051) (repomix output).*

### Query Multiple Rows, Map with `From<Row>` (`query_map`)

```rust
// async fn get_all_entities_map(client: &Client) -> Result<Vec<Entity>, Error> {
    let entities: Vec<Entity> = client
        .query_map("SELECT * FROM test", params!()) // Example: get all
        .await?;
    // for entity in &entities {
    //     println!("Fetched (query_map): {:?}", entity);
    // }
//     Ok(entities)
// }
```
*Source: [`examples/sqlite-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1061) (repomix output, adapted for clarity).*


## 5. Transactions (`txn`)

Execute multiple SQL statements atomically.

### Simple Transaction

```rust
// async fn multi_insert_txn(client: &Client) -> Result<(), Error> {
    let sql = "INSERT INTO test (id, num, description) VALUES ($1, $2, $3)";
    let results = client
        .txn([
            (sql, params!("id2", 345, "my description for 2. row")),
            (sql, params!("id3", 678, "my description for 3. row")),
            (sql, params!("id4", 999, "my description for 4. row")),
        ])
        .await?;

    for inner_res in results {
        let rows_affected = inner_res?;
        assert_eq!(rows_affected, 1); // Check each statement's result
    }
    // println!("Transaction successful.");
//     Ok(())
// }
```
*Source: [`examples/sqlite-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1073) (repomix output).*

### Transaction with `RETURNING` and `StmtIndex`

Pass values from `RETURNING` clauses of earlier statements in a transaction to later statements using `StmtIndex`.

```rust
// async fn parent_child_txn(client: &Client) -> Result<(), Error> {
    let insert_parent = "INSERT INTO parent (description) VALUES ($1) RETURNING id";
    let insert_child = "INSERT INTO child (parent_id, description) VALUES ($1, $2)";
    
    let results = client
        .txn([
            (insert_parent, params!("parent A")), // StmtIndex(0)
            (insert_parent, params!("parent B")), // StmtIndex(1)
            // Use ID from first parent insert (StmtIndex(0), column 0 which is 'id')
            (insert_child, params!(StmtIndex(0).column(0), "child A.1")),
            (insert_child, params!(StmtIndex(0).column(0), "child A.2")),
            // Use ID from second parent insert (StmtIndex(1), column named 'id')
            (
                insert_child,
                params!(StmtIndex(1).column("id"), "child B.1"),
            ),
        ])
        .await?;

    for inner_res in results {
        let rows_affected = inner_res?;
        assert_eq!(rows_affected, 1);
    }
    // println!("Parent-child transaction successful.");
//     Ok(())
// }
```
*Source: [`examples/sqlite-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1087) (repomix output).*
*Note: This feature was highlighted in the v0.6.0 changelog.*

## 6. Batch Execution (`batch`)

Execute a string containing multiple SQL statements separated by semicolons. This is simpler than `txn` for sequences of statements that don't require inter-statement value passing or complex rollback logic.

```rust
// async fn batch_example(client: &Client) -> Result<(), Error> {
    let mut results = client
        .batch(
            r#"
            INSERT INTO test (id, num, description) VALUES
                ('batch1', 1, "Batch desc 1"),
                ('batch2', 2, "Batch desc 2"),
                ('batch3', 3, "Batch desc 3");
            DELETE FROM test WHERE id = 'id4';
            "#,
        )
        .await?;

    // Results is a Vec<Result<usize, Error>>, one for each statement
    let rows_affected_insert = results.remove(0)?;
    assert_eq!(rows_affected_insert, 3);
    let rows_affected_delete = results.remove(0)?;
    assert_eq!(rows_affected_delete, 1);
    // println!("Batch execution successful.");
//     Ok(())
// }
```
*Source: [`examples/sqlite-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1106) (repomix output).*

These examples cover common use cases for SQL-only projects with Hiqlite v0.6.0. Refer to the `examples/sqlite-only` directory in the Hiqlite repository for the full runnable code.