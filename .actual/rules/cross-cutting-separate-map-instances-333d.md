# Use Map-Based Caching for MCP Server Configuration State: Separate Map Instances

These rules are ALWAYS ACTIVE for all configuration management modules handling MCP server state, including src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts.

### Rules

- **R-MAP-001** SHOULD: Separate Map instances SHOULD be maintained for each transport type (SSE, SHTTP, stdio) to isolate configuration domains.
- **R-MAP-002** MUST: Initialize separate Map instances for each transport type at function scope: `const existingSseServers = new Map(); const existingShttpServers = new Map(); const existingStdioServers = new Map();`
- **R-MAP-003** MUST: Use server name as key for stdio servers and URL as key for SSE/SHTTP servers.
- **R-MAP-004** MUST: Validate all configuration data through JSON.parse(content) or parseTOML(tomlContent) before Map.set() operations to ensure type safety.
- **R-MAP-005** MUST: Rebuild cache state from authoritative file system sources rather than maintaining long-lived cache instances.
- **R-MAP-006** SHOULD: Use Map.has() for existence checks before updates to implement upsert semantics and prevent unintended overwrites.

### Verify

```bash
# Verify Map instantiation for each transport type
grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations on cache instances
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify input validation before cache operations
grep -r 'JSON\.parse\|parseTOML' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts
```

**Accept when:**
- All MCP server configuration modules use Map data structures for caching server state, verified by presence of Map() instantiation and .set() operations.
- Input validation (JSON.parse or parseTOML) occurs before all cache insertion operations, verified by code flow analysis.
- Separate Map instances exist for each transport type (SSE, SHTTP, stdio) with appropriate key selection (name vs URL).
- Cache state is rebuilt from authoritative file system sources on each function invocation rather than persisted across calls.

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep commands must succeed and confirm Map-based caching patterns are present in configuration management modules.
</enforcement>