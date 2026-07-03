# Use Map-Based Caching for MCP Server Configuration State: Configuration Read Write

These rules are ALWAYS ACTIVE for all configuration management modules handling MCP server state, specifically files in `src/vscode/settings.ts` and `src/mcp/propagateOpenHandsMcp.ts` that manage server definitions across multiple transport types (SSE, SHTTP, stdio).

### Rules

- **R-CACHE-001** SHOULD: Configuration read/write functions (readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands) SHOULD rebuild cache state from authoritative sources on each invocation.
- **R-CACHE-002** MUST: Initialize separate Map instances for each transport type at function scope: `const existingSseServers = new Map(); const existingShttpServers = new Map(); const existingStdioServers = new Map();`
- **R-CACHE-003** MUST: Use server name as key for stdio servers and URL as key for SSE/SHTTP servers.
- **R-CACHE-004** MUST: Validate all configuration data through `JSON.parse(content)` or `parseTOML(tomlContent)` before `Map.set()` operations to ensure type safety.
- **R-CACHE-005** MUST: Use `Map.has()` for existence checks before updates to implement upsert semantics and prevent unintended overwrites.
- **R-CACHE-006** SHOULD: Implement write-through caching pattern where cache updates only succeed after successful file system persistence.

### Verify

```bash
# Verify Map instantiation for transport-specific caching
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
- Cache state is rebuilt from authoritative file system sources on each function invocation rather than maintained as long-lived instances

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep commands must return matches confirming Map-based caching patterns are present. Code review must validate that input validation precedes cache operations.
</enforcement>