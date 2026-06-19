# Use Map-Based In-Memory Cache for MCP Server Configuration State: Configuration Read Operations

These rules are ALWAYS ACTIVE for all MCP server configuration read/write operations in src/vscode/settings.ts, src/mcp/propagateOpenHandsMcp.ts, and related configuration management code that handles JSON (VSCode settings) and TOML (OpenHands config) file formats.

### Rules

- **R-CONFIG-001** MUST: Configuration read operations MUST populate the cache before any transformation or merge logic executes.
- **R-CONFIG-002** MUST: Use Map data structures for transient configuration state management with O(1) lookup and upsert operations.
- **R-CONFIG-003** MUST: Key Map instances by server name (STDIO transport) or URL (SSE/SHTTP transport) to enable automatic deduplication during configuration merges.
- **R-CONFIG-004** MUST: Initialize Map instances immediately after parsing configuration files (JSON.parse() or parseTOML()) and before any transformation logic.
- **R-CONFIG-005** MUST: Use Map.set() for all upsert operations to leverage automatic deduplication; avoid manual existence checks before insertion.
- **R-CONFIG-006** MUST: Maintain function-scoped cache lifecycle; do not introduce persistent or application-wide cache instances for MCP server configuration state.
- **R-CONFIG-007** MUST: Convert Map back to array or object structure using Array.from(map.values()) or Object.fromEntries() before serialization to JSON/TOML.
- **R-CONFIG-008** SHOULD: Use descriptive Map variable names that indicate transport type and purpose (e.g., existingSseServers, existingStdioServers, existingShttpServers).

### Verify

```bash
# Verify Map usage in configuration modules
grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations with appropriate key strategies
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|newServer\.name|url|serverDef\.url)'

# Verify Map usage in public API functions
grep -r 'readVSCodeSettings\|writeVSCodeSettings\|propagateMcpToOpenHands' src/ --include='*.ts' -A 20 | grep 'Map'
```

**Accept when:**
- All MCP server configuration read/write operations in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts use Map-based caching with appropriate key strategies (name for STDIO, URL for SSE/SHTTP).
- Map.set() operations are present for upserting server entries with keys derived from server name or URL.
- No persistent or application-wide cache instances are introduced for MCP server configuration state.
- Map instances are initialized immediately after file parsing and before transformation or merge logic.
- Map instances are function-scoped and converted back to serializable structures before JSON/TOML persistence.

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based caching patterns in configuration management code. All configuration read/write operations MUST be reviewed for compliance with R-CONFIG-001 through R-CONFIG-008 before approval.
</enforcement>