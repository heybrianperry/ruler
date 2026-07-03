# Use Map-Based In-Memory Cache for Server Configuration State: Server Configuration State

These rules are ALWAYS ACTIVE for all files matching server configuration state management, including src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts.

### Rules

- **R-CACHE-001** MUST: Server configuration state MUST be cached in Map data structures indexed by unique identifiers (server name or URL).
- **R-CACHE-002** MUST: Use server.name as the Map key for stdio-based servers and url for SSE/SHTTP servers to maintain consistent identity semantics.
- **R-CACHE-003** MUST: Initialize Map instances at the beginning of configuration read operations and populate via Map.set() during parsing.
- **R-CACHE-004** MUST: Convert Map to array or object format only during serialization using Array.from(existingServerMap.values()) or Object.fromEntries(existingServerMap).
- **R-CACHE-005** SHOULD: Extract Map-based cache logic into reusable utility functions to ensure consistent behavior across configuration management files.
- **R-CACHE-006** SHOULD: Implement explicit Map.delete() operations when servers are removed and add unit tests verifying cache cleanup.

### Verify

```bash
# Count Map instantiations for server configuration
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
Claude Code MUST NOT skip or defer verification. Violations detected by grep-based checks in CI pipeline or code review must be resolved before merge. Performance regression tests flagging degraded lookup times indicate non-Map implementations and require remediation.
</enforcement>