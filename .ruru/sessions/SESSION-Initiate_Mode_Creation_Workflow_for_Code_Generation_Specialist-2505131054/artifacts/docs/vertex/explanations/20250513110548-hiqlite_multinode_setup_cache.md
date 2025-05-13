To set up and configure `hiqlite` for a Rust project, including connecting to a potentially multi-node `rqlite` cluster, creating a client instance, and utilizing its cache feature, refer to the information synthesized from its official GitHub repository.

### Overview of `hiqlite`

`hiqlite` is a high-level, asynchronous Rust client for `rqlite`, a lightweight, distributed relational database that uses SQLite as its storage engine. `hiqlite` aims to simplify interactions with `rqlite` by providing a user-friendly API, connection pooling, optional caching, and automatic JSON serialization/deserialization.

**Source:** [https://github.com/sebadob/hiqlite](https://github.com/sebadob/hiqlite)

### Setting up `hiqlite` in a Rust Project

1.  **Add `hiqlite` as a dependency:**
    Include `hiqlite` in your `Cargo.toml` file. You'll also need `tokio` for the asynchronous runtime and `serde` for JSON serialization if you're working with custom types.

    ```toml
    [dependencies]
    hiqlite = "0.7" # Check for the latest version
    tokio = { version = "1", features = ["full"] }
    serde = { version = "1", features = ["derive"] }
    serde_json = "1"
    ```

### Configuring for a Multi-Node `rqlite` Setup

The `hiqlite` client is configured to connect to one or more `rqlite` nodes. `rqlite` itself handles the distributed consensus and data replication across its nodes. `hiqlite` facilitates connecting to such a cluster by allowing you to specify multiple node addresses.

1.  **Client Configuration with Multiple Nodes:**
    When creating a `HiqliteClient`, you can provide a list of host addresses for your `rqlite` nodes. The client will attempt to connect to these nodes. `rqlite` typically has a leader, and `hiqlite` will handle discovering the leader among the provided nodes for write operations. Read operations can often be served by any node, depending on the consistency level specified.

    The `HiqliteClient::new()` function accepts a `Vec<String>` of node URIs and a `HiqliteClientConfig`.

    **Source:** [https://github.com/sebadob/hiqlite - README section on Client Configuration](https://github.com/sebadob/hiqlite)

2.  **Leader Redirection and Node Discovery:**
    `hiqlite` is designed to handle `rqlite`'s leader redirection. If a request is sent to a non-leader node that requires leader processing (like writes), `rqlite` will respond with the leader's address. `hiqlite` should transparently handle this redirection and update its understanding of the current leader.

    The documentation mentions:
    *   "The client will automatically follow redirects from rqlite nodes, so you can provide a list of nodes and the client will find the leader."
    *   "Connections are managed in a pool, and the client tries to be smart about which node to pick (e.g. the leader for writes, a follower for reads if consistency allows)."

    **Source:** [https://github.com/sebadob/hiqlite - README](https://github.com/sebadob/hiqlite)

### Creating a Client Instance

You create a `HiqliteClient` instance using its `new` method, providing the node addresses and an optional configuration.

```rust
use hiqlite::{HiqliteClient, HiqliteClientConfig, HiqliteNodeConfig, Scheme};
use std::time::Duration;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Define the rqlite node addresses
    let nodes = vec![
        "http://localhost:4001".to_string(),
        "http://localhost:4003".to_string(), // Example of another node
        "http://localhost:4005".to_string(), // Example of yet another node
    ];

    // Basic client configuration
    let config = HiqliteClientConfig {
        node_config: HiqliteNodeConfig { // Configuration for each node
            scheme: Scheme::Http, // Or Scheme::Https if TLS is enabled on rqlite
            user: None, // Optional basic auth username
            pass: None, // Optional basic auth password
        },
        timeout: Duration::from_secs(10), // Request timeout
        consistency_level: Some("strong".to_string()), // Or "weak", "none"
        // Cache configuration is detailed in the next section
        cache_config: None,
        ..Default::default() // Use default for other fields like allow_redirects, etc.
    };

    // Create the client instance
    let client = HiqliteClient::new(nodes, config)?;

    println!("HiqliteClient created successfully.");

    // You can now use the client to interact with rqlite
    // Example: Ping the cluster
    match client.ping().await {
        Ok(true) => println!("Successfully pinged rqlite cluster."),
        Ok(false) => println!("Failed to ping rqlite cluster (leader not found or unhealthy)."),
        Err(e) => println!("Error pinging rqlite: {}", e),
    }

    Ok(())
}
```

**Key Configuration Aspects (`HiqliteClientConfig`):**

*   `nodes`: A `Vec<String>` containing URIs of the `rqlite` nodes (e.g., `http://localhost:4001`).
*   `node_config`: Contains `HiqliteNodeConfig` which specifies:
    *   `scheme`: `Scheme::Http` or `Scheme::Https`.
    *   `user`: Optional username for basic authentication.
    *   `pass`: Optional password for basic authentication.
*   `timeout`: Request timeout.
*   `consistency_level`: Can be `strong`, `weak`, or `none`. This affects read operations. `strong` ensures reads go to the leader.
*   `allow_redirects`, `max_redirects`: Control how the client handles HTTP redirects, particularly for leader discovery.
*   `cache_config`: Configuration for the optional caching layer (see below).

**Source:** [https://github.com/sebadob/hiqlite - README and examples](https://github.com/sebadob/hiqlite) and [docs.rs/hiqlite (for struct details)](https://docs.rs/hiqlite/latest/hiqlite/struct.HiqliteClientConfig.html)

### Using the Cache Feature

`hiqlite` includes an optional in-memory caching layer for `SELECT` queries. This can help reduce load on the `rqlite` cluster and improve performance for frequently accessed, rarely changing data.

1.  **Enabling and Configuring the Cache:**
    The cache is configured via the `cache_config` field in `HiqliteClientConfig`. You need to provide a `HiqliteClientCacheConfig` instance.

    ```rust
    use hiqlite::{HiqliteClient, HiqliteClientConfig, HiqliteNodeConfig, Scheme, HiqliteClientCacheConfig};
    use std::time::Duration;

    // ... (inside your async main function)

    let nodes = vec!["http://localhost:4001".to_string()];

    let cache_config = HiqliteClientCacheConfig {
        enabled: true,
        ttl_ms: Some(60_000), // Cache Time-To-Live in milliseconds (e.g., 60 seconds)
        max_size: Some(1000), // Maximum number of entries in the cache
    };

    let client_config = HiqliteClientConfig {
        node_config: HiqliteNodeConfig {
            scheme: Scheme::Http,
            user: None,
            pass: None,
        },
        timeout: Duration::from_secs(10),
        consistency_level: Some("strong".to_string()),
        cache_config: Some(cache_config), // Enable and configure the cache
        ..Default::default()
    };

    let client = HiqliteClient::new(nodes, client_config)?;
    ```

    **`HiqliteClientCacheConfig` fields:**
    *   `enabled`: A boolean to turn the cache on or off.
    *   `ttl_ms`: Optional `u64` specifying the Time-To-Live for cache entries in milliseconds. If `None`, entries live indefinitely until `max_size` is reached or they are manually invalidated (manual invalidation is not explicitly detailed as a direct API in the README, but cache respects TTL).
    *   `max_size`: Optional `usize` for the maximum number of query results to store in the cache. Uses an LRU (Least Recently Used) eviction policy when the cache is full.

    **Source:** [https://github.com/sebadob/hiqlite - README section on Caching](https://github.com/sebadob/hiqlite) and [docs.rs/hiqlite (for struct details)](https://docs.rs/hiqlite/latest/hiqlite/struct.HiqliteClientCacheConfig.html)

2.  **How the Cache Works:**
    *   When caching is enabled, `hiqlite` will store the results of `SELECT` queries.
    *   Subsequent identical `SELECT` queries (with the same statement and parameters) will be served from the cache if the entry is still valid (within TTL and cache size limits).
    *   The cache key is generated based on the SQL query string and its parameters.
    *   Operations other than `SELECT` (e.g., `INSERT`, `UPDATE`, `DELETE`) are not cached. The README does not explicitly state whether write operations invalidate relevant cache entries automatically; typically, simple client-side caches might not have this capability unless specifically designed for it. Users should be mindful of potential data staleness if the underlying data changes frequently and the TTL is long.

3.  **Using Cached Queries:**
    The caching mechanism is largely transparent. If enabled, `query_map`, `query_map_one`, `query_map_opt`, `query_json`, `query_json_one`, and `query_json_opt` methods that execute `SELECT` statements will automatically use the cache.

    ```rust
    use serde::Deserialize;
    use hiqlite::{HiqliteClient, HiqliteClientConfig, HiqliteNodeConfig, Scheme, HiqliteClientCacheConfig, Row}; // Assuming Row is used or a custom struct
    use std::time::Duration;

    #[derive(Deserialize, Debug)]
    struct MyData {
        id: i64,
        name: String,
    }

    #[tokio::main]
    async fn main() -> Result<(), Box<dyn std::error::Error>> {
        let nodes = vec!["http://localhost:4001".to_string()];
        let cache_config = HiqliteClientCacheConfig {
            enabled: true,
            ttl_ms: Some(5000), // Cache for 5 seconds
            max_size: Some(100),
        };
        let client_config = HiqliteClientConfig {
            node_config: HiqliteNodeConfig::default(), // Assumes http://localhost:4001
            cache_config: Some(cache_config),
            ..Default::default()
        };
        let client = HiqliteClient::new(nodes, client_config)?;

        // Ensure table exists (example setup)
        let _ = client.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)", &[]).await?;
        let _ = client.execute("INSERT OR IGNORE INTO users (id, name) VALUES (1, 'Alice')", &[]).await?;


        // First query: will fetch from rqlite and populate the cache
        println!("First query:");
        let user1: Option<MyData> = client.query_map_opt("SELECT id, name FROM users WHERE id = ?", &[1.into()]).await?;
        println!("User1: {:?}", user1);

        // Second query (identical): should be served from cache if within TTL
        println!("Second query (potentially cached):");
        let user2: Option<MyData> = client.query_map_opt("SELECT id, name FROM users WHERE id = ?", &[1.into()]).await?;
        println!("User2: {:?}", user2);

        // Wait for TTL to expire
        tokio::time::sleep(Duration::from_secs(6)).await;

        // Third query: should fetch from rqlite again as cache entry expired
        println!("Third query (after TTL expiry):");
        let user3: Option<MyData> = client.query_map_opt("SELECT id, name FROM users WHERE id = ?", &[1.into()]).await?;
        println!("User3: {:?}", user3);

        Ok(())
    }
    ```

    **Source:** [https://github.com/sebadob/hiqlite - README section on Caching and Usage Examples](https://github.com/sebadob/hiqlite)

### Summary

*   **Multi-Node Setup:** Configure `hiqlite` by providing a list of `rqlite` node addresses. The client handles leader discovery and redirection.
*   **Client Instance:** Create a `HiqliteClient` using `HiqliteClient::new()`, passing node URIs and a `HiqliteClientConfig` struct.
*   **Cache Feature:** Enable and configure caching via `HiqliteClientCacheConfig` within `HiqliteClientConfig`. It caches `SELECT` query results with configurable TTL and size, using an LRU eviction policy.

Always refer to the latest version of the `hiqlite` documentation on GitHub or docs.rs for the most up-to-date information and API details.