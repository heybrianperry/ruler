# Use Map-Based Caching for MCP Server Configuration State: Configuration Data Pass

These rules are ALWAYS ACTIVE for all configuration management modules handling MCP server state, including src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts.

### Rules

- **R-CACHE-001** MUST: Configuration data MUST pass through input validation (JSON.parse or parseTOML) before being stored in the cache layer.
- **R-CACHE-002** MUST: Use separate Map instances for each transport type (SSE, SHTTP, stdio) with appropriate key selection (server name for stdio, URL for SSE/SHTTP).
- **R-CACHE-003** MUST: Validate all configuration data through JSON.parse(content) or parseTOML(tomlContent) before Map.set() operations to ensure type safety.
- **R-CACHE-004** SHOULD: Rebuild cache state from authoritative file system sources (readVSCodeSettings, propagateMcpToOpenHands) rather than maintaining long-lived cache instances.
- **R-CACHE-005** SHOULD: Use Map.has() for existence checks before updates to implement upsert semantics and prevent unintended overwrites.
- **R-CACHE-006** MAY: Implement write-through caching pattern where cache updates only succeed after successful file system persistence to prevent cache state divergence.

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
- No direct object literal usage for server configuration caching is detected in configuration management modules

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration management modules handling MCP server state MUST comply with R-CACHE-001 through R-CACHE-006. Violations block code review and CI pipeline execution.
</enforcement>