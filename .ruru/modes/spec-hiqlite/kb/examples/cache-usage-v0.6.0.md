# Hiqlite: Cache Usage Examples (v0.6.0)

This document provides examples for using Hiqlite's in-memory caching features, based on version 0.6.0. Examples are primarily derived from `examples/cache-only/src/main.rs` and `hiqlite/src/client/cache.rs`.

## 1. Setup

### Dependencies

Ensure your `Cargo.toml` includes `hiqlite` with the `cache` feature and `hiqlite-macros`. The `strum` crate with the `derive` feature is also needed for the `CacheIndex` enum.

```toml
[dependencies]
hiqlite = { version = "0.6.0", default-features = false, features = ["cache"] } # Or path
hiqlite-macros = { version = "0.6.0" } # Or path

# Needed for the CacheIndex enum derivation
strum = { version = "0.27", features = ["derive"] }

serde = { version = "1.0", features = ["derive"] } # For serializing/deserializing cached values
tokio = { version = "1", features = ["rt-multi-thread"] }
# tracing and tracing-subscriber for logging (optional)
```

### Cache Enum and `CacheIndex` Implementation

Define an enum representing your different cache stores. This enum must derive `Debug` and `strum::EnumIter`. Then, implement the `hiqlite::cache_idx::CacheIndex` trait for it.

```rust
use hiqlite::cache_idx::CacheIndex;
use std::fmt::Debug; // Required by strum::EnumIter

#[derive(Debug, strum::EnumIter)]
enum Cache {
    StoreOne,
    StoreTwo,
}

impl CacheIndex for Cache {
    fn to_usize(self) -> usize {
        self as usize // Simple cast for basic enums
    }
}
```
*Source: Adapted from [`examples/cache-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:735) (repomix output). This reflects the v0.6.0 breaking change.*

### Value Struct

Define a struct for the values you'll be storing in the cache. It must derive `serde::Serialize` and `serde::Deserialize`.

```rust
use serde::{Deserialize, Serialize};
use std::fmt::Debug;

#[derive(Debug, PartialEq, Serialize, Deserialize)]
struct MyValue {
    pub id: String,
    pub data_field: i64,
    pub optional_info: Option<String>,
}
```
*Example adapted from [`examples/cache-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:745) (repomix output).*

## 2. Client Initialization with Cache

To use caching, start your Hiqlite node using `hiqlite::start_node_with_cache<YourCacheEnum>()`.

```rust
use hiqlite::{Client, Error, NodeConfig, start_node_with_cache};
// Assuming 'Cache' enum and 'MyValue' struct are defined as above

// async fn setup_node_with_cache() -> Result<Client, Error> {
    let config = NodeConfig::from_env_file("config"); // Or build programmatically
    let client = start_node_with_cache::<Cache>(config).await?;
//     Ok(client)
// }
```
*Source: [`examples/cache-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:758) (repomix output).*

## 3. Cache Operations

All cache operations are asynchronous and performed on the `Client` instance.

### Putting Values (`client.put`)

Stores a serializable value in the specified cache store with an optional Time-To-Live (TTL) in seconds.

```rust
// async fn put_example(client: &Client, key: &str, value: &MyValue) -> Result<(), Error> {
    // Put with a TTL of 60 seconds
    // client.put(Cache::StoreOne, key, value, Some(60)).await?;

    // Put without a TTL (persists until explicitly deleted or cache cleared)
    // client.put(Cache::StoreTwo, key, value, None).await?;
//     Ok(())
// }
```
*Source: [`hiqlite/src/client/cache.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1895) and [`examples/cache-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:766) (repomix output).*

### Getting Values (`client.get`)

Retrieves and deserializes a value from the cache. Returns `Ok(Some(V))` if found and deserialization succeeds, `Ok(None)` if not found or expired, and `Err` on other issues.

```rust
// async fn get_example(client: &Client, key: &str) -> Result<Option<MyValue>, Error> {
    // let maybe_value: Option<MyValue> = client.get(Cache::StoreOne, key).await?;
    // if let Some(value) = maybe_value {
    //     println!("Found value: {:?}", value);
    // } else {
    //     println!("Value not found or expired for key: {}", key);
    // }
//     Ok(maybe_value)
// }
```
*Source: [`hiqlite/src/client/cache.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1844) and [`examples/cache-only/src/main.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:768) (repomix output).*

### Deleting Values (`client.delete`)

Removes a specific key-value pair from a cache store.

```rust
// async fn delete_example(client: &Client, key: &str) -> Result<(), Error> {
    // client.delete(Cache::StoreOne, key).await?;
    // println!("Deleted key: {}", key);
//     Ok(())
// }
```
*Source: [`hiqlite/src/client/cache.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1937) (repomix output).*

### Clearing Cache Stores

*   **`client.clear_cache(YourCacheEnum::Variant)`**: Clears all entries from a specific cache store.
    ```rust
    // async fn clear_one_cache(client: &Client) -> Result<(), Error> {
        // client.clear_cache(Cache::StoreOne).await?;
        // println!("Cleared all entries from Cache::StoreOne");
    //     Ok(())
    // }
    ```
    *Source: [`hiqlite/src/client/cache.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1827) (repomix output).*

*   **`client.clear_cache_all()`**: Clears all entries from *all* defined cache stores.
    ```rust
    // async fn clear_all_caches(client: &Client) -> Result<(), Error> {
        // client.clear_cache_all().await?;
        // println!("Cleared all entries from all cache stores");
    //     Ok(())
    // }
    ```
    *Source: [`hiqlite/src/client/cache.rs`](./.ruru/docs/user-kb/repomix-output-sebadob-hiqlite.md:1840) (repomix output).*

These examples demonstrate the fundamental cache operations in Hiqlite v0.6.0. Remember that cache operations are distributed and replicated across the Raft cluster if multiple cache nodes are configured.