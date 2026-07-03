# Use Map-Based In-Memory Cache for Server Configuration State: Cache State Converted

These rules are ALWAYS ACTIVE for all files matching server configuration state management, including src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts.

### Rules

- **R-CACHE-001** SHOULD: Cache state SHOULD be converted to appropriate serialization formats (JSON, TOML) only during write operations.
- **R-CACHE-002** MUST: Use Map-based data structures for server configuration lookups to achieve O(1) time complexity.
- **R-CACHE-003** MUST: Use server.name as the Map key for stdio-based servers and url for SSE/SHTTP servers to maintain consistent identity semantics.
- **R-CACHE-004** SHOULD: Convert Map to array or object format only during serialization using Array.from(existingServerMap.values()) or Object.fromEntries(existingServerMap).
- **R-CACHE-005** MUST: Implement explicit Map.delete() operations when servers are removed to prevent memory leaks.
- **R-CACHE-006** SHOULD: Extract Map-based cache logic into reusable utility functions to ensure consistent behavior across configuration management files.

### Verify

```bash
# Count Map instantiations for server configuration
grep -r 'new Map()' src/ | grep -E '(Server|server)' | wc -l

# Count Map.set() operations on server cache variables
grep -r '\.set\(' src/ | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)' | wc -l

# Count linear Array.find() patterns for server lookups
grep -r 'Array\.find\(' src/ | grep -E '(server|Server)' | wc -l
```

**Accept when:**
- Map-based cache patterns are detected in at least 2 files handling server configuration (verified by grep showing Map instantiation and .set() operations)
- No linear Array.find() patterns exist for server lookups in configuration management code paths
- All server configuration updates use Map.set() with consistent key strategies (name for stdio, url for SSE/SHTTP)
- Map instances are properly cleared with Map.delete() when servers are removed

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for server configuration state management code. Violations must be flagged during code review and CI pipeline checks must fail if Map-based patterns are not detected or if Array.find() patterns are introduced in server configuration paths.
</enforcement>