# Adopt Map-Based In-Memory Cache for Server Configuration State: Map Set Operations

These rules are ALWAYS ACTIVE for all server configuration management code that performs read-modify-write operations on server definitions across multiple transport types (SSE, SHTTP, STDIO).

### Rules

- **R-MAP-001** SHOULD: Map.set() operations SHOULD be used to insert or update server entries, leveraging native Map semantics for automatic deduplication by key.
- **R-MAP-002** MUST: Maintain separate Map instances per transport type (SSE, SHTTP, STDIO) to prevent key collisions between different server identity semantics.
- **R-MAP-003** MUST: Use server.name as the key for STDIO transport servers and server.url (or serverDef.url) as the key for SSE and SHTTP transport servers.
- **R-MAP-004** MUST: Initialize Map instances immediately after parsing configuration files using new Map() to ensure clean state.
- **R-MAP-005** MUST: Convert Map values back to arrays using Array.from(existingMap.values()) before serializing to storage.
- **R-MAP-006** SHOULD: Enforce function-scoped cache instances to prevent memory leaks from inadvertent closure capture or global scope retention.

### Verify

```bash
# Count Map instances used for server configuration caching
grep -r 'new Map<' src/ --include='*.ts' | grep -E '(Server|server)' | wc -l

# Count Map.set() operations on server configuration maps
grep -r '\.set\(' src/ --include='*.ts' | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)' | wc -l

# Verify Map usage with server identity keys
grep -r 'Map.*(name|url).*server' src/ --include='*.ts' -A 5 | grep -c 'set('
```

**Accept when:**
- Verification commands confirm at least 2 Map instances are used for server configuration caching across the codebase.
- Code review confirms separate Map instances exist for each server transport type (SSE, SHTTP, STDIO) with appropriate key selection.
- Manual inspection of configuration management functions shows Map.set() operations occur after file parsing and before modifications.
- All server configuration caching follows the read-cache-modify-write pattern with function-scoped Map lifecycle.

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration management modules must be inspected to confirm Map-based caching pattern adherence before accepting changes to server state management.
</enforcement>