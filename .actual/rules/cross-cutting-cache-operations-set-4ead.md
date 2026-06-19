# Use Map-Based Caching for MCP Server Configuration State: Cache Operations Set

These rules are ALWAYS ACTIVE for all configuration management modules handling MCP server state, including src/vscode/settings.ts, src/mcp/propagateOpenHandsMcp.ts, and any functions managing server definitions across multiple transport types (SSE, SHTTP, stdio).

### Rules

- **R-CACHE-001** MUST: Cache operations (set, get, has) MUST use the Map.set() method to ensure atomic key-value updates.
- **R-CACHE-002** MUST: Validate all configuration data through JSON.parse(content) or parseTOML(tomlContent) before Map.set() operations to ensure type safety.
- **R-CACHE-003** MUST: Initialize separate Map instances for each transport type at function scope: const existingSseServers = new Map(); const existingShttpServers = new Map(); const existingStdioServers = new Map().
- **R-CACHE-004** MUST: Use server name as key for stdio servers and URL as key for SSE/SHTTP servers to maintain consistent key selection across transport types.
- **R-CACHE-005** SHOULD: Rebuild cache state from authoritative file system sources (readVSCodeSettings, propagateMcpToOpenHands) rather than maintaining long-lived cache instances.
- **R-CACHE-006** SHOULD: Use Map.has() for existence checks before updates to implement upsert semantics and prevent unintended overwrites.
- **R-CACHE-007** MAY: Implement write-through caching pattern where cache updates only succeed after successful file system persistence to prevent cache state divergence.

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
- All MCP server configuration modules use Map data structures for caching server state, verified by presence of Map() instantiation and .set() operations.
- Input validation (JSON.parse or parseTOML) occurs before all cache insertion operations, verified by code flow analysis.
- Separate Map instances exist for each transport type (SSE, SHTTP, stdio) with appropriate key selection (name for stdio, URL for SSE/SHTTP).
- Cache operations use atomic Map.set() method calls rather than direct object property assignment.
- No direct object literal usage for server configuration caching is present in configuration management modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep commands MUST return results matching the expected patterns. Code review MUST block merge if configuration caching does not follow Map-based pattern. CI pipeline MUST fail if verification commands do not find expected Map usage patterns.
</enforcement>