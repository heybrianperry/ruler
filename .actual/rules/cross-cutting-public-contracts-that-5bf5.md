# Use Map-Based In-Memory Cache for MCP Server Configuration State: Public Contracts That

These rules are ALWAYS ACTIVE for all MCP server configuration read/write operations in src/vscode/settings.ts, src/mcp/propagateOpenHandsMcp.ts, and public API functions that manipulate MCP server configurations (readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands, transformRulerToAugmentMcp).

### Rules

- **R-MCP-CACHE-001** SHOULD: Public API contracts that manipulate MCP server configurations SHOULD use the Map-based cache pattern for consistency across all file formats and transport types.
- **R-MCP-CACHE-002** MUST: Use descriptive Map variable names that indicate transport type and purpose (e.g., existingSseServers, existingStdioServers, existingShttpServers) to improve code clarity.
- **R-MCP-CACHE-003** MUST: Initialize Map instances immediately after parsing configuration files (JSON.parse() or parseTOML()) and before any transformation logic to ensure cache is populated.
- **R-MCP-CACHE-004** MUST: Use Map.set() for all upsert operations to leverage automatic deduplication; avoid manual existence checks before insertion.
- **R-MCP-CACHE-005** MUST: Key Map entries by server name for STDIO transport type and by URL for SSE/SHTTP transport types to enable proper deduplication.
- **R-MCP-CACHE-006** MUST: Convert Map back to array or object structure using Array.from(map.values()) or Object.fromEntries() before serialization to JSON/TOML.
- **R-MCP-CACHE-007** MUST: Maintain function-scoped cache lifecycle; do not introduce persistent or application-wide cache instances for MCP server configuration state.
- **R-MCP-CACHE-008** MUST NOT: Introduce external caching dependencies (Redis, LRU cache libraries) for transient configuration state management.

### Verify

```bash
# Verify Map usage in configuration modules
grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations with appropriate keys
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|newServer\.name|url|serverDef\.url)'

# Verify Map usage in public API functions
grep -r 'readVSCodeSettings\|writeVSCodeSettings\|propagateMcpToOpenHands' src/ --include='*.ts' -A 20 | grep 'Map'

# Verify no persistent cache instances
grep -r 'const.*Cache.*=.*new Map()' src/ --include='*.ts' | grep -v 'function\|const.*=.*function'
```

**Accept when:**
- All MCP server configuration read/write operations in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts use Map-based caching with appropriate key strategies (name for STDIO, URL for SSE/SHTTP).
- Map.set() operations are present for upserting server entries with descriptive variable names indicating transport type.
- Map instances are initialized immediately after parsing configuration files and before transformation logic.
- Map instances are converted back to array or object structures before JSON/TOML serialization using Array.from(map.values()) or Object.fromEntries().
- No persistent or application-wide cache instances are introduced for MCP server configuration state.
- All public API functions (readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands, transformRulerToAugmentMcp) follow the Map-based caching pattern.

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based caching pattern compliance in MCP server configuration operations. All configuration read/write code MUST be reviewed against these rules before acceptance.
</enforcement>