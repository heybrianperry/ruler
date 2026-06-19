# Use Map-Based In-Memory Cache for Server Configuration State: Implementations Use Separate

These rules are ALWAYS ACTIVE for all files matching server configuration state management, including src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts, that require fast lookup and update operations on server entries identified by unique keys.

### Rules

- **R-MAP-001** MAY: Implementations MAY use separate Map instances for different server transport types (stdio, SSE, SHTTP).
- **R-MAP-002** MUST: Use server.name as the Map key for stdio-based servers and url for SSE/SHTTP servers to maintain consistent identity semantics.
- **R-MAP-003** MUST: Initialize Map instances at the beginning of configuration read operations and populate via Map.set() during parsing.
- **R-MAP-004** MUST: Convert Map to array or object format only during serialization using Array.from(existingServerMap.values()) or Object.fromEntries(existingServerMap).
- **R-MAP-005** SHOULD: Extract Map-based cache logic into reusable utility functions to ensure consistent behavior across multiple configuration management files.
- **R-MAP-006** MUST: Implement explicit Map.delete() operations when servers are removed to prevent memory leaks.
- **R-MAP-007** MUST: Avoid linear Array.find() patterns for server lookups in configuration management code paths; use Map-based lookups instead.

### Verify

```bash
# Count Map instantiations for server configuration
grep -r 'new Map()' src/ | grep -E '(Server|server)' | wc -l

# Count Map.set() operations on server configuration maps
grep -r '\.set\(' src/ | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)' | wc -l

# Verify no linear Array.find() patterns exist for server lookups
grep -r 'Array\.find\(' src/ | grep -E '(server|Server)' | wc -l
```

**Accept when:**
- Map-based cache patterns are detected in at least 2 files handling server configuration (verified by grep showing Map instantiation and .set() operations)
- No linear Array.find() patterns exist for server lookups in configuration management code paths
- All server configuration updates use Map.set() with consistent key strategies (name for stdio, url for SSE/SHTTP)
- Map instances are properly initialized before configuration read operations
- Serialization uses Array.from() or Object.fromEntries() for Map-to-format conversion

<enforcement>
Claude Code MUST NOT skip or defer verification. All server configuration state management code MUST comply with these Map-based caching rules. Violations detected by grep-based verification commands in CI pipeline MUST be resolved before merge.
</enforcement>