# Use Map-Based Caching for MCP Server Configuration State: Mcp Server Configuration

These rules are ALWAYS ACTIVE for all configuration management modules handling MCP server state, specifically files in `src/vscode/settings.ts` and `src/mcp/propagateOpenHandsMcp.ts` that manage server definitions across multiple transport types (SSE, SHTTP, stdio).

### Rules

- **R-MCP-001** MUST: All MCP server configuration state MUST be cached in Map data structures keyed by unique identifiers (server name for stdio, URL for SSE/SHTTP).
- **R-MCP-002** MUST: Validate all configuration data through JSON.parse(content) or parseTOML(tomlContent) before Map.set() operations to ensure type safety.
- **R-MCP-003** MUST: Initialize separate Map instances for each transport type at function scope: const existingSseServers = new Map(); const existingShttpServers = new Map(); const existingStdioServers = new Map().
- **R-MCP-004** MUST: Use server name as key for stdio servers and URL as key for SSE/SHTTP servers.
- **R-MCP-005** SHOULD: Rebuild cache state from authoritative file system sources rather than maintaining long-lived cache instances.
- **R-MCP-006** SHOULD: Use Map.has() for existence checks before updates to implement upsert semantics and prevent unintended overwrites.
- **R-MCP-007** MAY: Implement write-through caching pattern where cache updates only succeed after successful file system persistence.

### Verify

```bash
# Verify Map instantiation for all transport types
grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations on cache instances
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify input validation before cache operations
grep -r 'JSON\.parse\|parseTOML' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts
```

**Accept when:**
- All MCP server configuration modules use Map data structures for caching server state, verified by presence of Map() instantiation and .set() operations
- Input validation (JSON.parse or parseTOML) occurs before all cache insertion operations, verified by code flow analysis
- Separate Map instances exist for each transport type (SSE, SHTTP, stdio) with appropriate key selection (name vs URL)
- Cache state is rebuilt from authoritative file system sources rather than maintained across function invocations

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting changes to configuration management modules. Code review MUST block merge if configuration caching does not follow Map-based pattern. CI pipeline MUST fail if grep verification commands do not find expected Map usage patterns.
</enforcement>