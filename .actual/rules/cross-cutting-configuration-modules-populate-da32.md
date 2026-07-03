# Adopt Map-Based In-Memory Cache for Server Configuration Management: Configuration Modules Populate

These rules are ALWAYS ACTIVE for configuration modules performing server configuration read-transform-write cycles, including functions in `src/vscode/settings.ts` and `src/mcp/propagateOpenHandsMcp.ts` that handle VSCode settings and OpenHands MCP server definitions.

### Rules

- **R-CONFIG-MAP-001** MUST: Configuration modules MUST populate Maps from parsed file content (JSON.parse() or parseTOML()) before performing transformation or merge operations.
- **R-CONFIG-MAP-002** MUST: Use server.name as key for STDIO servers and server.url (or serverDef.url) as key for SSE/SHTTP servers to maintain consistent keying strategy.
- **R-CONFIG-MAP-003** MUST: Initialize Map instances at the start of configuration read operations with clear naming convention using 'existing' prefix (existingServerMap, existingSseServers, existingShttpServers, existingStdioServers).
- **R-CONFIG-MAP-004** MUST: Convert Map back to array before serialization using Array.from(mapInstance.values()) or [...mapInstance.values()].
- **R-CONFIG-MAP-005** SHOULD: Scope Map variables to function execution lifetime to prevent stale state and ensure thread-safe intermediate state management.
- **R-CONFIG-MAP-006** SHOULD: Document keying strategy in type definitions and function documentation to prevent inconsistent key usage across server types.

### Verify

```bash
# Verify Map-based cache variables are present in configuration modules
grep -r 'existingServerMap\|existingSseServers\|existingShttpServers\|existingStdioServers' src/ --include='*.ts'

# Verify Map.set() operations are used in configuration merge logic
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts

# Verify Map initialization occurs at start of configuration functions
grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts
```

**Accept when:**
- Grep commands identify Map-based cache variables (existingServerMap, existingSseServers, existingShttpServers, existingStdioServers) in both src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts
- Map.set() operations are present in configuration merge logic for server deduplication and insertion
- Map initialization (new Map()) occurs at the start of configuration read-transform-write functions
- Server keying strategy is consistently applied: URL-based servers use URL as key, STDIO servers use name as key
- Map-to-array conversion is performed before persistence operations using Array.from() or spread operator

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep commands MUST execute successfully and return results matching the expected patterns before accepting configuration module changes.
</enforcement>