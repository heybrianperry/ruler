# Use Map-Based Caching for MCP Server Configuration State: Cache State Synchronized

These rules are ALWAYS ACTIVE for all configuration management modules handling MCP server state, including src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts.

### Rules

- **R-CACHE-001** MUST: Cache state MUST be synchronized with file system persistence operations through FileSystemUtils to prevent data loss.
- **R-CACHE-002** MUST: Initialize separate Map instances for each transport type (SSE, SHTTP, stdio) at function scope.
- **R-CACHE-003** MUST: Use server name as key for stdio servers and URL as key for SSE/SHTTP servers.
- **R-CACHE-004** MUST: Validate all configuration data through JSON.parse() or parseTOML() before Map.set() operations to ensure type safety.
- **R-CACHE-005** MUST: Rebuild cache state from authoritative file system sources rather than maintaining long-lived cache instances.
- **R-CACHE-006** SHOULD: Use Map.has() for existence checks before updates to implement upsert semantics and prevent unintended overwrites.

### Verify

```bash
# Verify Map-based caching is used for server configuration state
grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations are present for cache updates
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify input validation occurs before cache operations
grep -r 'JSON\.parse\|parseTOML' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts
```

**Accept when:**
- All MCP server configuration modules use Map data structures for caching server state, verified by presence of Map() instantiation and .set() operations
- Input validation (JSON.parse or parseTOML) occurs before all cache insertion operations, verified by code flow analysis
- Separate Map instances exist for each transport type (SSE, SHTTP, stdio) with appropriate key selection (name vs URL)
- Cache state is rebuilt from authoritative file system sources on each function invocation
- Map.has() is used for existence checks before updates

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep commands must succeed and confirm Map-based caching patterns are present. Code review must validate that input validation precedes all cache operations.
</enforcement>