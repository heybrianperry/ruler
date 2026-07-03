# Adopt Map-Based In-Memory Cache for Server Configuration State: Cache Instances Scoped

These rules are ALWAYS ACTIVE for all server configuration management code that performs read-modify-write operations on configuration files, particularly in modules handling VSCode settings and OpenHands MCP propagation across multiple transport types (SSE, SHTTP, STDIO).

### Rules

- **R-CACHE-001** SHOULD: Cache instances SHOULD be scoped to the function performing the read-modify-write operation to avoid unintended state sharing across invocations.
- **R-CACHE-002** MUST: Use separate Map instances for each server transport type (SSE, SHTTP, STDIO) to prevent key collisions.
- **R-CACHE-003** MUST: Use `server.name` as the Map key for STDIO transport servers and `server.url` (or `serverDef.url`) as the Map key for SSE and SHTTP transport servers.
- **R-CACHE-004** MUST: Initialize Map instances immediately after parsing configuration files using `new Map()` to ensure clean state.
- **R-CACHE-005** MUST: Populate the cache by iterating over existing server arrays and calling `existingMap.set(key, server)` for each entry.
- **R-CACHE-006** MUST: Convert Map values back to arrays using `Array.from(existingMap.values())` before serializing to storage.

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
- Manual inspection of configuration management functions shows `Map.set()` operations occur after file parsing and before modifications.
- All Map instances are function-scoped and not captured in closures or global scope beyond their intended lifecycle.

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration management code introducing or modifying server caching patterns MUST be verified against these rules before approval.
</enforcement>