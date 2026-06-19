# Standardize Map-Based In-Memory Cache for Server Configuration State: Cache Operations Use

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The codebase manages server configuration state across multiple modules (VSCode settings and OpenHands MCP propagation) that require fast, indexed access to server definitions by name or URL
- Server configuration data is read from persistent storage (JSON and TOML files), transformed in memory, and written back, requiring an intermediate representation that supports efficient lookup and mutation
- The pattern emerged in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts, both of which use Map.set() operations to maintain server registries indexed by server.name or url
- The public API contracts (VSCodeSettings, AugmentMcpServer, readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands) expose server configuration operations that depend on this cache layer for correctness

## Problem Statement

Server configuration state must be efficiently queried, updated, and synchronized across multiple file formats (JSON, TOML) and integration points (VSCode, OpenHands), requiring a consistent in-memory representation that supports indexed access while maintaining referential integrity during read-modify-write cycles.

## Decision

1. MUST: Cache operations MUST use Map.set() to insert or update server entries, ensuring idempotent upsert semantics

## Policy Block

- MUST Cache operations MUST use Map.set() to insert or update server entries, ensuring idempotent upsert semantics

In scope:
- Server configuration modules that read from or write to VSCode settings.json
- Server configuration modules that read from or write to OpenHands config.toml
- Public API functions that expose server registry operations (readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands)
- In-memory transformations of server definitions between file formats

Out of scope:
- Persistent storage implementations (file system, database)
- Server runtime state or connection pooling
- Client-side caching of server responses
- Distributed cache systems or shared memory

## Rationale

- The evidence shows consistent use of Map.set() operations across two independent modules (src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts) with 91% confidence, indicating an established pattern rather than coincidental implementation
- Map data structures provide O(1) lookup and upsert performance for server configurations indexed by name or URL, which is critical for read-modify-write workflows that must preserve existing entries while adding or updating specific servers
- The pattern coordinates with security.input_validation (JSON.parse, parseTOML) and paradigm.concurrency_model (async functions), suggesting the cache layer serves as a validated, mutable intermediate representation between untrusted file input and trusted API output
- The api.public.contracts facet detection (VSCodeSettings, AugmentMcpServer, readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands) confirms this cache layer is architecturally significant as it underpins the public API surface

## Consequences

Positive:
- O(1) lookup and upsert performance for server configuration operations, enabling efficient read-modify-write cycles without full array scans
- Idempotent upsert semantics via Map.set() prevent duplicate server entries and simplify merge logic when synchronizing configurations
- Clear separation between file I/O, parsing, cache management, and API contracts improves testability and maintainability
- Consistent indexing strategy (name for stdio, URL for SSE/SHTTP) enforces type-specific identity semantics at the cache layer

Negative:
- Map-based caches are ephemeral and must be rebuilt on every read operation, introducing memory allocation overhead for large server registries
- The pattern does not address concurrent modification scenarios if multiple processes or threads attempt to update the same configuration files simultaneously
- Developers must manually maintain consistency between Map keys and server object properties (name, url), as there is no type-level enforcement of this invariant
- The cache layer adds an intermediate representation that must be serialized back to JSON/TOML, increasing the complexity of the write path

## Alternatives

- Use plain JavaScript objects with bracket notation for server lookups instead of Map instances (rejected)
  Rejected because: Plain objects do not provide the same semantic clarity for upsert operations and lack built-in methods for iteration and size queries that Map provides. The evidence shows deliberate use of Map.set() rather than object property assignment.
  When valid: For simple, static configuration lookups where upsert semantics and iteration are not required
- Maintain server configurations as arrays and use Array.find() for lookups (rejected)
  Rejected because: Array-based lookups have O(n) complexity and require manual deduplication logic during upserts. The evidence shows Map.set() usage specifically to avoid these performance and correctness issues.
  When valid: For small server registries (< 10 entries) where lookup performance is not critical and insertion order must be preserved
- Introduce a persistent cache layer (SQLite, Redis) to avoid rebuilding Maps on every read (deferred)
  Rejected because: No evidence of persistent cache infrastructure in the detected modules. This would add significant complexity and external dependencies.
  When valid: If profiling reveals that Map reconstruction overhead is a performance bottleneck, or if concurrent access patterns require transactional semantics

## Risks

- Concurrent modifications to configuration files by multiple processes could lead to lost updates or inconsistent state if Map-based caches are not synchronized with file system state
  Mitigation: Implement file locking or atomic write-rename patterns when persisting configuration changes. Document that configuration updates should be serialized through a single process.
  Owner: engineering team
- Memory usage scales linearly with the number of server configurations, which could become problematic for deployments with hundreds or thousands of servers
  Mitigation: Monitor memory usage in production environments. If server registries exceed 1000 entries, consider pagination or lazy-loading strategies for configuration reads.
  Owner: engineering team
- Type safety is not enforced between Map keys and server object properties, allowing runtime errors if server.name or url fields are missing or malformed
  Mitigation: Add input validation in readVSCodeSettings and propagateMcpToOpenHands to verify that server objects contain required key fields before Map.set() operations. Consider TypeScript type guards.
  Owner: engineering team

## Implementation Notes

- When implementing server configuration modules, initialize separate Map instances for each server transport type (stdio, SSE, SHTTP) to enforce type-specific indexing strategies
- Use Map.set(key, value) for all server upsert operations to ensure idempotent semantics. Avoid manual existence checks followed by insertion.
- Populate Map-based caches immediately after parsing configuration files (JSON.parse, parseTOML) and before any transformation or merge logic
- When serializing Map-based caches back to configuration files, use Map.values() or Array.from(map.values()) to extract server objects, ensuring the output format matches the expected JSON/TOML schema

## Continuation Context


Verify commands:
- grep -r 'Map.set' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|url)' | wc -l
- grep -r 'new Map<' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | wc -l
- npm test -- --testPathPattern='(settings|propagateOpenHandsMcp)' --testNamePattern='cache|Map'

Accept when:
- Grep commands confirm at least 4 Map.set() operations using server.name or url as keys across the two detected modules
- Grep commands confirm at least 2 Map instantiations in server configuration modules
- Unit tests for readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands verify that Map-based caches correctly handle upsert, lookup, and serialization scenarios

## Enforcement

- Verified by: Code review checklist item: Verify that server configuration modules use Map.set() for upsert operations
- Verified by: Static analysis: Grep for Map.set() usage in modules that import VSCodeSettings, AugmentMcpServer, or propagateMcpToOpenHands types
- Verified by: Unit test coverage: Ensure tests verify Map-based cache behavior for server configuration operations
- Violation handling: Code review feedback: Request refactoring to use Map.set() instead of array-based or object-based lookups
- Violation handling: If performance issues arise from non-Map implementations, file a technical debt ticket to refactor to the standard pattern
- Violation handling: Document exceptions in module-level comments if alternative data structures are required for specific use cases
- Exception process: Submit a design review document explaining why Map-based caching is unsuitable for the specific use case
- Exception process: Obtain approval from the architecture review board or lead engineer
- Exception process: Document the exception in the module's README or inline comments, including rationale and alternative approach