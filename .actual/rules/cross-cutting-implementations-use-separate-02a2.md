# Use Map-Based In-Memory Cache for MCP Server Configuration State: Implementations Use Separate

These rules are ALWAYS ACTIVE for all MCP server configuration read/write operations in `src/vscode/settings.ts`, `src/mcp/propagateOpenHandsMcp.ts`, and related configuration management code that handles JSON (VSCode settings) and TOML (OpenHands config) file formats.

### Rules

- **R-CACHE-001** MAY: Implementations MAY use separate Map instances for different transport types (existingSseServers, existingShttpServers, existingStdioServers) to enforce type-specific keying strategies.
- **R-CACHE-002** MUST: Use descriptive Map variable names that indicate transport type and purpose (e.g., existingSseServers, existingStdioServers) to improve code clarity.
- **R-CACHE-003** MUST: Initialize Map instances immediately after parsing configuration files and before any transformation logic to ensure cache is populated.
- **R-CACHE-004** MUST: Use Map.set() for all upsert operations to leverage automatic deduplication; avoid manual existence checks before insertion.
- **R-CACHE-005** MUST: Convert Map back to array or object structure using Array.from(map.values()) or Object.fromEntries() before serialization to JSON/TOML.
- **R-CACHE-006** MUST NOT: Introduce persistent or application-wide cache instances for MCP server configuration state; maintain function-scoped cache lifecycle.
- **R-CACHE-007** MUST NOT: Use plain JavaScript objects with bracket notation for caching server configurations; use Map for guaranteed key iteration order and prototype pollution safety.

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
- Map.set() operations are present for upserting server entries with correct keying strategies.
- No persistent or application-wide cache instances are introduced for MCP server configuration state.
- Map instances are function-scoped and initialized immediately after parsing configuration files.
- Conversion from Map to serializable structures (Array.from() or Object.fromEntries()) occurs before JSON/TOML persistence.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All configuration management code must be reviewed against R-CACHE-001 through R-CACHE-007 before acceptance.
</enforcement>