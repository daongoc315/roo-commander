# Hiqlite: Full-Feature Walkthrough Examples (v0.6.0)

This document provides examples demonstrating the combined usage of Hiqlite's features, including SQLite database operations, in-memory caching, distributed locks (`dlock`), and listen/notify capabilities. These examples are based on Hiqlite version 0.6.0 and are primarily derived from the `examples/walkthrough/src/main.rs` file in the Hiqlite repository.

## 1. Setup

The setup involves defining entities, migrations, and a cache enum similar to those in the SQL-only and cache-only examples.

### Dependencies

Ensure your `Cargo.toml` includes `hiqlite` with all relevant features (`cache`, `dlock`, `listen_notify`, `shutdown-handle`) and `hiqlite-macros`.

```toml
[dependencies]
hiqlite = { version = "0.6.0", features = ["cache", "dlock", "listen_notify", "shutdown-handle"] } # Or path
hiqlite-macros = { version = "0.6.0" } # Or path

strum = { version = "0.27", features = ["derive"] } # For CacheIndex enum
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1", features = ["rt-multi-thread", "fs"] }
# clap, tracing, etc. as needed by the example
```

### Entity and Cache Definitions

```rust
use hiqlite::Row;
use hiqlite::cache_idx::CacheIndex;
use serde::{Deserialize, Serialize};
use std::fmt::Debug;

// For database interactions
#[derive(Debug, PartialEq, Serialize, Deserialize, Clone)] // Clone might be needed for notify
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

// For cache
#[derive(Debug, strum::EnumIter)]
enum Cache {
    One,
    Two,
}

impl CacheIndex for Cache {
    fn to_usize(self) -> usize {
        self as usize
    }
}
```
*Source: Adapted from [`examples/walkthrough/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1382) (repomix output).*

### Migrations

Define and embed SQL migrations as shown in previous examples.
```rust
use hiqlite_macros::embed::*;

#[derive(Embed)]
#[folder = "migrations"] // Assuming migrations are in a "migrations" directory
struct Migrations;
```

## 2. Client Initialization (Full Features)

Start a Hiqlite node with cache capabilities using `start_node_with_cache`. This client will support all features if the corresponding feature flags were enabled during compilation.

```rust
use hiqlite::{start_node_with_cache, Client, Error, NodeConfig, Node};
// ... other necessary imports ...

// fn node_config(node_id: u64, nodes: Vec<Node>) -> NodeConfig { /* ... */ }
// fn test_nodes() -> Vec<Node> { /* ... */ }

// async fn initialize_full_feature_client() -> Result<Client, Error> {
    // let node_id = 1; // Example node ID
    // let mut config = node_config(node_id, test_nodes());
    // config.data_dir = format!("data/node_{}", node_id).into();
    // // ... other NodeConfig settings ...

    let client = start_node_with_cache::<Cache>(config).await?;
    // Give the client some time to initialize, especially in a cluster
    // tokio::time::sleep(std::time::Duration::from_secs(3)).await;
//     Ok(client)
// }
```
*Source: [`examples/walkthrough/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1447) (repomix output).*

## 3. Combined Feature Usage Example Flow

The `examples/walkthrough/src/main.rs` demonstrates a sequence of operations. Below is a conceptual flow combining these features:

```rust
use hiqlite::{params, Client, Error}; // Assuming Entity, Cache, Migrations are defined
// use std::time::Duration;
// use tokio::time;

// async fn run_walkthrough(client: Client) -> Result<(), Error> {
    // 1. Apply Migrations
    println!("Applying migrations...");
    // client.migrate::<Migrations>().await?;

    // 2. Basic Database Operations (Execute, Insert, Query)
    println!("Performing database operations...");
    // client.execute("DELETE FROM test", params!()).await?;
    // client.execute(
    //     "INSERT INTO test (id, num, description) VALUES ($1, $2, $3)",
    //     params!("id_walkthrough", 789, "Full feature example row"),
    // ).await?;
    // let entity_db: Entity = client.query_as_one("SELECT * FROM test WHERE id = $1", params!("id_walkthrough")).await?;
    // println!("Fetched from DB: {:?}", entity_db);

    // 3. Cache Operations
    println!("Performing cache operations...");
    let cache_key = "walkthrough_cache_key";
    let cache_value = "This is a cached string for the walkthrough".to_string();
    // client.put(Cache::One, cache_key, &cache_value, Some(60)).await?; // TTL 60s
    // let retrieved_cache_value: Option<String> = client.get(Cache::One, cache_key).await?;
    // assert_eq!(retrieved_cache_value, Some(cache_value.clone()));
    // println!("Fetched from Cache: {:?}", retrieved_cache_value.unwrap());

    // 4. Distributed Lock (dlock)
    println!("Testing distributed lock...");
    // {
    //     println!("Acquiring lock...");
    //     let _lock = client.lock("my_walkthrough_lock_key").await?;
    //     println!("Lock acquired. Performing locked operation (e.g., critical read/write)...");
    //     // Simulate work
    //     // time::sleep(Duration::from_secs(2)).await;
    //     println!("Releasing lock (on drop)...");
    // } // _lock is dropped here, releasing the distributed lock

    // 5. Listen/Notify
    println!("Testing listen/notify...");
    let notification_payload = Entity {
        id: "notify_id".to_string(),
        num: 101,
        description: Some("Listen/Notify payload".to_string()),
    };

    // Task to listen for a notification
    // let listener_client = client.clone();
    // tokio::spawn(async move {
    //     println!("Listener: Waiting for notification...");
    //     match listener_client.listen::<Entity>().await {
    //         Ok(msg) => println!("Listener: Received notification: {:?}", msg),
    //         Err(e) => eprintln!("Listener: Error listening: {:?}", e),
    //     }
    // });

    // Give the listener a moment to start up
    // time::sleep(Duration::from_millis(100)).await;

    // Send a notification
    // println!("Notifier: Sending notification: {:?}", notification_payload);
    // client.notify(&notification_payload).await?;
    
    // Allow time for notification to be processed
    // time::sleep(Duration::from_secs(1)).await;

    // 6. Shutdown Handle
    // (Setup is typically done at client initialization, wait at the end of main)
    // let mut shutdown_handle = client.shutdown_handle()?;
    // println!("All operations complete. Waiting for shutdown.");
    // shutdown_handle.wait().await?;
//     Ok(())
// }
```
*This is a conceptual combination based on operations found in [`examples/walkthrough/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1315) (repomix output lines 1483-1586).*

This walkthrough demonstrates how different features of Hiqlite can be used together in an application. For detailed, runnable code, refer to the `examples/walkthrough/` directory in the official Hiqlite repository.