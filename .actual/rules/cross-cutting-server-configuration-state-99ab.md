# Adopt Map-Based In-Memory Cache for Server Configuration State: Server Configuration State

These rules are ALWAYS ACTIVE for all configuration management modules that handle server state caching during read-modify-write operations, particularly in VSCode settings and OpenHands MCP propagation code paths.

### Rules

- **R-CACHE-001** MUST: Server configuration state MUST be cached in Map instances keyed by server identity (name for STDIO, URL for SSE/SHTTP) during configuration read-modify-write operations.
- **R-CACHE-002** MUST: Initialize Map instances immediately after parsing configuration files using `new Map()` to ensure clean state.
- **R-CACHE-003** MUST: Use server.name as the key for STDIO transport servers and server.url (or serverDef.url) as the key for SSE and SHTTP transport servers.
- **R-CACHE-004** MUST: Maintain separate Map instances per transport type (SSE, SHTTP, STDIO) to prevent key collisions.
- **R-CACHE-005** MUST: Populate the cache by iterating over existing server arrays and calling `existingMap.set(key, server)` for each entry.
- **R-CACHE-006** MUST: Convert Map values back to arrays using `Array.from(existingMap.values())` before serializing to storage.
- **R-CACHE-007** SHOULD: Document key selection requirements in code comments to ensure correct identity semantics across different server transport types.

### Verify

```bash
# Verify Map instances are used for server configuration caching
grep -r 'new Map<' src/ --include='*.ts' | grep -E '(Server|server)' | wc -l

# Verify Map.set() operations on server configuration caches
grep -r '\.set\(' src/ --include='*.ts' | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)' | wc -l

# Verify Map usage with appropriate key selection
grep -r 'Map.*(name|url).*server' src/ --include='*.ts' -A 5 | grep -c 'set('
```

**Accept when:**
- Verification commands confirm at least 2 Map instances are used for server configuration caching across the codebase.
- Code review confirms separate Map instances exist for each server transport type (SSE, SHTTP, STDIO) with appropriate key selection.
- Manual inspection of configuration management functions shows Map.set() operations occur after file parsing and before modifications.
- Array.from(existingMap.values()) pattern is used to convert cached state back to arrays before serialization.

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration management code introducing server state caching MUST adhere to R-CACHE-001 through R-CACHE-007. Violations detected in code review result in request for refactoring to align with established pattern.
</enforcement>