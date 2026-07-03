# Use Map-Based In-Memory Cache for Server Configuration State: Map Based Caches

These rules are ALWAYS ACTIVE for configuration synchronization logic in server management code, particularly when handling read-modify-write cycles across multiple configuration file formats (JSON for VSCode, TOML for OpenHands).

### Rules

- **R-CACHE-001** SHOULD: Map-based caches SHOULD be scoped to individual function invocations rather than persisted as module-level state, ensuring transactional isolation.
- **R-CACHE-002** MUST: Use Map data structures for server configuration lookups to achieve O(1) average-case complexity for insertion and lookup operations.
- **R-CACHE-003** MUST: Initialize Map instances at the start of configuration synchronization functions by iterating over existing server arrays and using appropriate keys (server.name for STDIO, server.url for SSE/SHTTP).
- **R-CACHE-004** MUST: Use separate Map instances per server type to enforce correct key selection at compile time and prevent cache misses or incorrect deduplication.
- **R-CACHE-005** SHOULD: Convert Map instances back to arrays using Array.from(map.values()) before serializing to JSON or TOML formats.
- **R-CACHE-006** SHOULD: Extract Map construction and conversion logic into reusable utility functions to reduce code duplication across settings.ts and propagateOpenHandsMcp.ts.

### Verify

```bash
# Verify Map instances are present in configuration synchronization functions
grep -r 'Map<.*>' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations are used for cache updates
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map instances are constructed within function scope
grep -r 'new Map' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts
```

**Accept when:**
- Grep commands identify Map instances named existingServerMap, existingSseServers, existingShttpServers, or existingStdioServers in both src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts
- Map.set() operations are present in configuration synchronization functions for updating server cache entries
- Map instances are constructed within function scope rather than as module-level globals
- Separate Map instances are used for different server types (STDIO vs SSE/SHTTP) with appropriate key selection
- Array-to-Map-to-array conversion logic is present before serialization to JSON or TOML formats

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration synchronization functions MUST use Map-based caching with appropriate key selection. Static analysis MUST detect Array.find() usage in hot paths and flag deviations from Map-based caching. Performance testing MUST verify configuration operations complete within acceptable latency thresholds.
</enforcement>