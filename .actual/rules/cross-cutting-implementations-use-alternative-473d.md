# Adopt Map-Based In-Memory Cache for Server Configuration State: Implementations Use Alternative

These rules are ALWAYS ACTIVE for all configuration management modules that handle server state caching during read-modify-write operations, including VSCode settings and OpenHands MCP propagation code.

### Rules

- **R-CACHE-001** MAY: Implementations MAY use alternative key-value data structures if they provide equivalent O(1) lookup and automatic deduplication semantics.
- **R-CACHE-002** MUST: Initialize Map instances immediately after parsing configuration files using `new Map()` to ensure clean state.
- **R-CACHE-003** MUST: Use server.name as the key for STDIO transport servers and server.url (or serverDef.url) as the key for SSE and SHTTP transport servers.
- **R-CACHE-004** MUST: Maintain separate Map instances per transport type (SSE, SHTTP, STDIO) to prevent key collisions between different server types.
- **R-CACHE-005** MUST: Populate the cache by iterating over existing server arrays and calling `existingMap.set(key, server)` for each entry.
- **R-CACHE-006** MUST: Convert Map values back to arrays using `Array.from(existingMap.values())` before serializing to storage.
- **R-CACHE-007** SHOULD: Enforce function-scoped cache instances and conduct code reviews to verify proper scope management to prevent memory leaks.
- **R-CACHE-008** SHOULD: Document key selection requirements in code comments to ensure correct identity semantics across different server transport types.

### Verify

```bash
# Count Map instances used for server configuration caching
grep -r 'new Map<' src/ --include='*.ts' | grep -E '(Server|server)' | wc -l

# Count Map.set() operations on server configuration maps
grep -r '\.set\(' src/ --include='*.ts' | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)' | wc -l

# Verify Map usage with name/url keys for servers
grep -r 'Map.*(name|url).*server' src/ --include='*.ts' -A 5 | grep -c 'set('
```

**Accept when:**
- Verification commands confirm at least 2 Map instances are used for server configuration caching across the codebase
- Code review confirms separate Map instances exist for each server transport type (SSE, SHTTP, STDIO) with appropriate key selection
- Manual inspection of configuration management functions shows Map.set() operations occur after file parsing and before modifications
- Array.from(existingMap.values()) pattern is used to convert cached Maps back to arrays before serialization

<enforcement>
Clause Code MUST NOT skip or defer verification. All configuration management modules introducing or modifying server state caching MUST be reviewed against these rules. Violations detected in code review result in request for refactoring to align with established pattern. Exception requests must document specific performance or functional requirements that Map-based caching cannot satisfy and require approval from module owner.
</enforcement>