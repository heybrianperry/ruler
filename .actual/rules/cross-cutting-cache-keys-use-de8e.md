# Use Map-Based In-Memory Cache for Server Configuration State: Cache Keys Use

These rules are ALWAYS ACTIVE for all server configuration management code in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts that handles in-memory caching of server entries.

### Rules

- **R-CACHE-001** MUST: Cache keys MUST use server.name for stdio-based servers and url for SSE/SHTTP-based servers
- **R-CACHE-002** MUST: Initialize Map instances at the beginning of configuration read operations using `const existingServerMap = new Map()`
- **R-CACHE-003** MUST: Populate Map instances via Map.set() during parsing, never via direct array manipulation
- **R-CACHE-004** MUST: Convert Map to array or object format only during serialization using `Array.from(existingServerMap.values())` or `Object.fromEntries(existingServerMap)`
- **R-CACHE-005** SHOULD: Extract Map-based cache logic into reusable utility functions to ensure consistent behavior across configuration files
- **R-CACHE-006** MUST: Implement explicit Map.delete() operations when servers are removed to prevent memory leaks
- **R-CACHE-007** MUST NOT: Use Array.find() for server lookups in configuration management code paths
- **R-CACHE-008** MUST NOT: Use plain objects with bracket notation as a substitute for Map-based caching in server configuration code

### Verify

```bash
# Count Map instantiations for server-related caching
grep -r 'new Map()' src/ | grep -E '(Server|server)' | wc -l

# Count Map.set() operations on server cache variables
grep -r '\.set\(' src/ | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)' | wc -l

# Verify no linear Array.find() patterns exist for server lookups
grep -r 'Array\.find\(' src/ | grep -E '(server|Server)' | wc -l
```

**Accept when:**
- Map-based cache patterns are detected in at least 2 files handling server configuration (verified by grep showing Map instantiation and .set() operations)
- No linear Array.find() patterns exist for server lookups in configuration management code paths
- All server configuration updates use Map.set() with consistent key strategies (name for stdio, url for SSE/SHTTP)
- Map instances are properly cleared with Map.delete() when servers are removed

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All server configuration caching code MUST conform to Map-based patterns with consistent key selection strategies. Violations detected by grep-based verification commands MUST be resolved before code acceptance.
</enforcement>