# Adopt Map-Based In-Memory Cache for Server Configuration State: Each Server Transport

These rules are ALWAYS ACTIVE for all server configuration management code that performs read-modify-write cycles on server definitions across multiple transport types (SSE, SHTTP, STDIO).

### Rules

- **R-CACHE-001** MUST: Each server transport type (SSE, SHTTP, STDIO) MUST maintain a separate Map instance to prevent namespace collisions between different transport protocols.
- **R-CACHE-002** MUST: Use server.name as the key for STDIO transport servers and server.url (or serverDef.url) as the key for SSE and SHTTP transport servers.
- **R-CACHE-003** MUST: Initialize Map instances immediately after parsing configuration files using new Map() to ensure clean state.
- **R-CACHE-004** MUST: Populate the cache by iterating over existing server arrays and calling existingMap.set(key, server) for each entry.
- **R-CACHE-005** MUST: Convert Map values back to arrays using Array.from(existingMap.values()) before serializing to storage.
- **R-CACHE-006** SHOULD: Enforce function-scoped cache instances to prevent memory leaks from inadvertent closure capture or global scope retention.

### Verify

```bash
# Verify Map instances are used for server configuration caching
grep -r 'new Map<' src/ --include='*.ts' | grep -E '(Server|server)' | wc -l

# Verify Map.set() operations on server configuration maps
grep -r '\.set\(' src/ --include='*.ts' | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)' | wc -l

# Verify Map usage with correct key selection patterns
grep -r 'Map.*(name|url).*server' src/ --include='*.ts' -A 5 | grep -c 'set('
```

**Accept when:**
- Verification commands confirm at least 2 Map instances are used for server configuration caching across the codebase.
- Code review confirms separate Map instances exist for each server transport type (SSE, SHTTP, STDIO) with appropriate key selection.
- Manual inspection of configuration management functions shows Map.set() operations occur after file parsing and before modifications.
- No array-based lookups with Array.find() are used for server configuration reconciliation in place of Map instances.

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration management code introducing or modifying server caching patterns MUST be verified against these rules before acceptance.
</enforcement>