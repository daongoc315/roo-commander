+++
id = "spec-hiqlite-kb-lookup-rule-v1"
title = "Knowledge Base Lookup Rule for spec-hiqlite"
context_type = "rules"
scope = "Mode-specific knowledge base access and utilization guidance for Hiqlite operations"
target_audience = ["spec-hiqlite"]
granularity = "rule"
status = "active"
last_updated = "2025-05-13" # Today's date
# version = "1.0"
related_context = [
    ".ruru/modes/spec-hiqlite/kb/README.md",
    ".ruru/modes/spec-hiqlite/spec-hiqlite.mode.md"
]
tags = ["kb", "lookup", "mode-specific", "spec-hiqlite", "hiqlite", "database", "rqlite", "rust"]
# relevance = "High: Ensures spec-hiqlite leverages its specialized knowledge."
+++

# Knowledge Base (KB) Lookup Rule for `spec-hiqlite`

**Applies To:** `spec-hiqlite` mode

**Rule:**

Before attempting any task involving Hiqlite, **ALWAYS** consult the dedicated Knowledge Base (KB) directory for this mode located at:

`./.ruru/modes/spec-hiqlite/kb/`

This KB contains crucial information about Hiqlite's features, Rust client usage, and best practices for interacting with `rqlite` clusters.

**Procedure:**

1.  **Identify Hiqlite-Specific Keywords & Concepts:** Determine the key aspects of the task related to Hiqlite. This includes, but is not limited to:
    *   **Core Operations:** Database connection (`HiqliteClient`, `HiqliteClientConfig`), CRUD operations (`execute`, `query_map`, `query_map_one`, `query_map_opt`), data types, SQL syntax for SQLite.
    *   **Distributed Features:** Raft consensus implications, leader redirection, node discovery, consistency levels (`strong`, `weak`, `none`).
    *   **Schema Management:** `CREATE TABLE`, `CREATE INDEX`, `ALTER TABLE` specific to SQLite via Hiqlite.
    *   **Performance:** Client-side caching (`HiqliteClientCacheConfig`), indexing strategies.
    *   **Transactions:** `BEGIN`, `COMMIT`, `ROLLBACK` usage with `execute`.
    *   **Client Configuration:** `HiqliteNodeConfig`, authentication (Basic Auth), TLS (`Scheme::Https`).
    *   **Error Handling:** Common issues and their resolutions.
    *   **Advanced Topics:** Backup/recovery considerations (though likely managed at the `rqlite` level), security (client-to-node).

2.  **Scan KB Systematically:**
    *   Start with the main KB [`README.md`](./.ruru/modes/spec-hiqlite/kb/README.md:1) for an overview and links to key documents.
    *   Review relevant subdirectories such as:
        *   `fundamentals/` (e.g., [`database-overview-summary.md`](./.ruru/modes/spec-hiqlite/kb/fundamentals/database-overview-summary.md:1))
        *   `client/` (e.g., [`setup-connection-summary.md`](./.ruru/modes/spec-hiqlite/kb/client/setup-connection-summary.md:1))
        *   `schema/` (e.g., [`data-definition-schema-summary.md`](./.ruru/modes/spec-hiqlite/kb/schema/data-definition-schema-summary.md:1))
        *   `usage/` (e.g., [`basic-crud-summary.md`](./.ruru/modes/spec-hiqlite/kb/usage/basic-crud-summary.md:1), [`querying-data-summary.md`](./.ruru/modes/spec-hiqlite/kb/usage/querying-data-summary.md:1))
        *   `performance/` (e.g., [`indexing-summary.md`](./.ruru/modes/spec-hiqlite/kb/performance/indexing-summary.md:1))
        *   `transactions/` (e.g., [`transactions-summary.md`](./.ruru/modes/spec-hiqlite/kb/transactions/transactions-summary.md:1))
        *   `security/` (e.g., [`security-access-control-summary.md`](./.ruru/modes/spec-hiqlite/kb/security/security-access-control-summary.md:1))
        *   `operations/` (e.g., [`backup-recovery-summary.md`](./.ruru/modes/spec-hiqlite/kb/operations/backup-recovery-summary.md:1))
    *   Look for specific code examples, configuration details, and explanations of Hiqlite's behavior, especially how it wraps `rqlite` functionalities.

3.  **Apply Hiqlite Knowledge:**
    *   Prioritize information from the KB for Hiqlite-specific syntax (Rust client methods, parameter types like `hiqlite::Value`), configuration structures, and operational best practices.
    *   Understand that Hiqlite is a Rust client for `rqlite`, which uses SQLite. Your solutions should reflect this layered architecture.
    *   Pay attention to asynchronous patterns (`async/await`) as `hiqlite` is an async Rust client.

4.  **If KB is Empty/Insufficient:**
    *   If the KB does not cover a specific aspect of the task:
        *   Clearly state that the information is not available in the `spec-hiqlite` KB.
        *   Proceed by applying general knowledge of `rqlite`, SQLite, distributed systems (Raft), and Rust database clients, making reasonable assumptions.
        *   Explicitly mention any assumptions made due to the lack of specific Hiqlite KB guidance.
        *   Recommend that the missing information be added to the KB for future reference.

**Rationale:** This rule ensures that `spec-hiqlite` leverages its specialized, curated knowledge for consistent, accurate, and effective operation when dealing with Hiqlite. Adhering to this rule promotes maintainability and allows for future knowledge expansion, leading to higher quality solutions.