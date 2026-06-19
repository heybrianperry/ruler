# Use Map-Based In-Memory Cache for MCP Server Configuration State: Cache Operations Use

These rules are ALWAYS ACTIVE for all MCP server configuration read/write operations, including files in `src/vscode/settings.ts`, `src/mcp/propagateOpenHandsMcp.ts`, and public API functions that manage configuration state across multiple file formats (JSON, TOML).

### Rules

- **R-CACHE-001** MUST: Cache operations MUST use `Map.set()` for upsert semantics to ensure deduplication of server entries by key.
- **R-CACHE-002** MUST: Map instances MUST be function-scoped and initialized immediately after parsing configuration files, before any transformation logic.
- **R-CACHE-003** MUST: Use descriptive Map variable names that indicate transport type and purpose (e.g., `existingSseServers`, `existingStdioServers`, `existingShttpServers`).
- **R-CACHE-004** MUST: Key Map entries by server name for STDIO transport type and by URL for SSE/SHTTP transport types.
- **R-CACHE-005** MUST: Convert Map back to array or object structure using `Array.from(map.values())` or `Object.fromEntries()` before serialization to JSON/TOML.
- **R-CACHE-006** SHOULD: Avoid manual existence checks before `Map.set()` insertion to leverage automatic deduplication semantics.
- **R-CACHE-007** MUST NOT: Introduce persistent or application-wide cache instances for MCP server configuration state.
- **R-CACHE-008** MUST NOT: Use plain JavaScript objects with bracket notation as a substitute for Map-based caching in configuration management.

### Verify

```bash
# Verify Map usage in configuration modules
grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations with appropriate keys
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|newServer\.name|url|serverDef\.url)'

# Verify Map usage in public API functions
grep -r 'readVSCodeSettings\|writeVSCodeSettings\|propagateMcpToOpenHands' src/ --include='*.ts' -A 20 | grep 'Map'
```

**Accept when:**
- All MCP server configuration read/write operations in `src/vscode/settings.ts` and `src/mcp/propagateOpenHandsMcp.ts` use Map-based caching with appropriate key strategies (name for STDIO, URL for SSE/SHTTP).
- `Map.set()` operations are present for upserting server entries with no manual existence checks preceding insertion.
- Map instances are function-scoped and initialized immediately after configuration file parsing.
- Descriptive Map variable names indicating transport type are used throughout the codebase.
- Map instances are converted back to array or object structures before JSON/TOML serialization.
- No persistent or application-wide cache instances are introduced for MCP server configuration state.
- Unit tests validate deduplication behavior through `Map.set()` semantics.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All MCP server configuration operations MUST comply with Map-based caching requirements before code review approval.
</enforcement>