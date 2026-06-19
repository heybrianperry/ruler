# Use Map-Based In-Memory Cache for Server Configuration State: Configuration Read Operations

These rules are ALWAYS ACTIVE for all files matching the configured scope: server configuration management code in `src/vscode/settings.ts`, `src/mcp/propagateOpenHandsMcp.ts`, and related configuration read/write operations.

### Rules

- **R-CONFIG-001** SHOULD: Configuration read operations SHOULD populate the cache before performing transformations or lookups.
- **R-CONFIG-002** MUST: Use Map data structures (not plain arrays or objects) for server configuration state indexed by unique keys (server name or URL).
- **R-CONFIG-003** MUST: Use server.name as the Map key for stdio-based servers and url for SSE/SHTTP servers to maintain consistent identity semantics.
- **R-CONFIG-004** SHOULD: Initialize Map instances at the beginning of configuration read operations and populate via Map.set() during parsing.
- **R-CONFIG-005** SHOULD: Convert Map to array or object format only during serialization using Array.from(existingServerMap.values()) or Object.fromEntries(existingServerMap).
- **R-CONFIG-006** MUST: Implement explicit Map.delete() operations when servers are removed to prevent memory leaks.
- **R-CONFIG-007** MAY: Extract Map-based cache logic into reusable utility functions to ensure consistent behavior across multiple files.

### Verify

```bash
# Count Map instantiations for server configuration
grep -r 'new Map()' src/ | grep -E '(Server|server)' | wc -l

# Count Map.set() operations on server cache variables
grep -r '\.set\(' src/ | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)' | wc -l

# Count Array.find() patterns for server lookups (should be minimal/zero)
grep -r 'Array\.find\(' src/ | grep -E '(server|Server)' | wc -l
```

**Accept when:**
- Map-based cache patterns are detected in at least 2 files handling server configuration (verified by grep showing Map instantiation and .set() operations)
- No linear Array.find() patterns exist for server lookups in configuration management code paths
- All server configuration updates use Map.set() with consistent key strategies (name for stdio, url for SSE/SHTTP)
- Map.delete() operations are present for server removal scenarios

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for configuration read operations. Violations must be flagged during code review and CI pipeline checks must fail if Map-based patterns are not detected or if Array.find() patterns are introduced in server configuration code paths.
</enforcement>