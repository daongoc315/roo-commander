# Hiqlite Client Setup & Connection (v0.6.0)

This document outlines how to initialize and configure the Hiqlite `Client` for interacting with a Hiqlite node or cluster, based on version 0.6.0.

## 1. Overview

The Hiqlite `Client` provides the primary interface for applications to communicate with Hiqlite. Initialization differs based on whether you are connecting to a remote Hiqlite instance/cluster or creating/connecting to a local instance (typically for testing or embedded scenarios).

## 2. Connecting to a Remote Hiqlite Node/Cluster

Use `Client::remote()` to establish a connection to one or more remote Hiqlite nodes.

**Parameters:**

*   `nodes`: `Vec<String>` - A list of node API addresses (e.g., `"127.0.0.1:8101"`).
*   `tls`: `bool` - Whether to use TLS for the API connection.
*   `tls_no_verify`: `bool` - If `true`, disables TLS certificate verification (use with caution, primarily for testing).
*   `api_secret`: `String` - The API secret required to authenticate with the Hiqlite nodes.
*   `with_proxy`: `bool` - If `true`, the client assumes it's connecting through a proxy and will not attempt leader discovery among the provided nodes. If `false`, it will attempt to find and connect to the leader.

**Example:**

From `examples/bench/src/main.rs`:

```rust
use hiqlite::Client;

// Assuming 'opts' is a struct containing connection parameters
// async fn setup_remote_client(opts: RemoteOptions) -> Result<Client, hiqlite::Error> {
//     let client = Client::remote(
//         opts.nodes,         // e.g., vec!["127.0.0.1:8101".to_string(), "127.0.0.1:8102".to_string()]
//         opts.tls,           // e.g., false
//         opts.tls_no_verify, // e.g., false
//         opts.api_secret,    // e.g., "SuperSecureApiSecret".to_string()
//         true,               // with_proxy (can be true or false based on setup)
//     )
//     .await?;
//     Ok(client)
// }
```
*Source: [`examples/bench/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:528) in repomix output.*
*Definition: [`hiqlite/src/client/create.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:2086) in repomix output.*

## 3. Creating/Connecting to a Local Hiqlite Node

For local instances (often used in single-node setups, embedded use cases, or integration tests), the `Client` is typically obtained by starting a Hiqlite node directly within the application. The `Client::new_local()` method is `crate`-internal and is invoked by helper functions like `hiqlite::start_node()` or `hiqlite::start_node_with_cache()`.

**Using `hiqlite::start_node()` (SQLite features only):**

This function initializes a Hiqlite node with SQLite capabilities and returns a `Client` connected to it.

**Parameters:**

*   `config`: `NodeConfig` - Configuration for the Hiqlite node.

**Example:**

From `examples/sqlite-only/src/main.rs`:

```rust
use hiqlite::{NodeConfig, Client}; // Assuming Error is also in scope

// async fn setup_local_node() -> Result<Client, hiqlite::Error> {
//     let config = NodeConfig::from_env_file("config"); // Or build programmatically
//     let client = hiqlite::start_node(config).await?;
//     Ok(client)
// }
```
*Source: [`examples/sqlite-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1025) in repomix output.*

**Using `hiqlite::start_node_with_cache()` (SQLite and Cache features):**

This function initializes a Hiqlite node with both SQLite and in-memory caching capabilities, returning a `Client`.

**Parameters:**

*   `config`: `NodeConfig` - Configuration for the Hiqlite node.
*   Requires a generic type `C` that implements `CacheIndex` (e.g., an enum defining cache stores).

**Example:**

From `examples/bench/src/main.rs`:

```rust
use hiqlite::{start_node_with_cache, Client, NodeConfig};
use hiqlite::cache_idx::CacheIndex; // For CacheIndex trait
// Assuming Error is also in scope

// Define your cache enum
// #[derive(Debug, strum::EnumIter)]
// enum MyCache {
//     CacheStore1,
//     CacheStore2,
// }
//
// impl CacheIndex for MyCache {
//     fn to_usize(self) -> usize {
//         self as usize
//     }
// }

// async fn setup_local_node_with_cache() -> Result<Client, hiqlite::Error> {
//     let mut config = NodeConfig::default(); // Populate as needed
//     // ... configure 'config' ...
//     let client = start_node_with_cache::<MyCache>(config).await?;
//     Ok(client)
// }
```
*Source: [`examples/bench/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:575) and [`hiqlite/src/start.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:14084) (for `start_node_inner` which `start_node_with_cache` calls) in repomix output.*

The `Client::new_local()` function itself takes several internal parameters related to the node's `AppState`, TLS configuration, communication channels, and shutdown handles. Application developers will typically not call `Client::new_local()` directly but will use `start_node` or `start_node_with_cache`.

## 4. Configuration (`NodeConfig`)

Both local node startup functions (`start_node`, `start_node_with_cache`) rely on a `NodeConfig` struct. This struct contains various settings for the node, including:

*   `node_id`: `u64`
*   `nodes`: `Vec<Node>` (list of all nodes in the cluster, including their API and Raft addresses)
*   `data_dir`: Path for storing data.
*   `secret_api`: `String`
*   `secret_raft`: `String`
*   `tls_api`: Optional `ServerTlsConfig`
*   `tls_raft`: Optional `ServerTlsConfig`
*   And many other Raft and application-specific settings.

`NodeConfig` can be created programmatically or loaded from an environment file (e.g., `NodeConfig::from_env_file("config")`). Refer to the `NodeConfig` definition in [`hiqlite/src/config.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:12507) and example configuration files for more details.