# Hiqlite Security & Access Control

The provided context for `hiqlite` primarily details client-side connection and interaction with an `rqlite` cluster. Security features are mostly discussed in the context of connecting the `hiqlite` client to `rqlite` nodes.

**1. Client-to-Node Authentication:**

*   **Basic Authentication:** `hiqlite` supports connecting to `rqlite` nodes using HTTP Basic Authentication.
    *   This is configured in `HiqliteClientConfig` within the `node_config: HiqliteNodeConfig` field.
    *   `user`: Optional `String` for the username.
    *   `pass`: Optional `String` for the password.

    **Example Snippet from Configuration:**
    ```rust
    // Part of HiqliteClientConfig
    node_config: HiqliteNodeConfig {
        scheme: Scheme::Http, // Or Scheme::Https
        user: Some("myuser".to_string()),
        pass: Some("mypassword".to_string()),
    }
    ```

**2. Secure Transport (TLS/HTTPS):**

*   `hiqlite` supports connecting to `rqlite` nodes over a secure channel using HTTPS.
*   This is configured by setting the `scheme` field in `HiqliteNodeConfig` to `Scheme::Https`.
    ```rust
    // Part of HiqliteClientConfig
    node_config: HiqliteNodeConfig {
        scheme: Scheme::Https,
        // ... other fields like user/pass if applicable
    }
    ```
    This implies that the `rqlite` cluster itself must be configured to serve traffic over HTTPS with appropriate TLS certificates.

**Features Not Explicitly Detailed in Provided `hiqlite` Context:**

The provided documents focusing on `hiqlite` (the Rust client) do not go into detail on the following server-side (`rqlite` or SQLite) security mechanisms:

*   **User Roles and Permissions/Privileges (Database-Level):** Standard SQLite has a very limited concept of users and permissions (mainly filesystem-based for the database file itself in a non-distributed context). `rqlite` might layer additional access control, but this is not detailed in the `hiqlite` client documentation provided.
*   **Row-Level Security (RLS):** Not mentioned.
*   **Network Access Controls (beyond client auth/TLS):** Specific firewall rules or network policies for the `rqlite` cluster would be configured at the infrastructure level, not directly through the `hiqlite` client.
*   **Data Encryption at Rest:** While SQLite supports extensions for encryption (like SEE or SQLCipher), the provided `hiqlite` context does not mention built-in support or configuration for these through the client. This would typically be a feature of the `rqlite` build or underlying SQLite configuration.

**Summary:**
For `hiqlite`, security considerations from the provided documents focus on:
*   Authenticating the client to `rqlite` nodes using Basic Authentication.
*   Ensuring secure communication using HTTPS/TLS.

More advanced database-level security features like granular user permissions, roles, or RLS would depend on the capabilities of the `rqlite` server and are not directly managed or configured through the `hiqlite` client API as per the supplied information.