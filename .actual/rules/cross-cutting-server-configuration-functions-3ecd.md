# Adopt Map-Based In-Memory Cache for Server Configuration Management: Server Configuration Functions

These rules are ALWAYS ACTIVE for server configuration functions performing read-transform-write cycles across VSCode settings and OpenHands MCP integrations, specifically in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts.

### Rules

- **R-CACHE-001** SHOULD: Server configuration functions SHOULD use consistent naming patterns (existingServerMap, existingSseServers, existingStdioServers) to indicate Map-based cache usage.
- **R-CACHE-002** MUST: Use server.name as key for STDIO servers and server.url (or serverDef.url) as key for SSE/SHTTP servers to maintain consistent keying strategy.
- **R-CACHE-003** MUST: Initialize Map instances at the start of configuration read operations and populate from parsed file content.
- **R-CACHE-004** MUST: Convert Map back to array before serialization using Array.from(existingServerMap.values()) or [...existingServerMap.values()].
- **R-CACHE-005** SHOULD: Scope Map variables with 'existing' prefix to clearly indicate they represent current state from persistent storage.

### Verify

```bash
# Verify Map-based cache variables are present in configuration modules
grep -r 'existingServerMap\|existingSseServers\|existingShttpServers\|existingStdioServers' src/ --include='*.ts'

# Verify Map.set() operations are used in configuration merge logic
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts

# Verify Map initialization occurs at function start
grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts
```

**Accept when:**
- Grep commands identify Map-based cache variables (existingServerMap, existingSseServers, existingShttpServers, existingStdioServers) in both src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts
- Map.set() operations are present in configuration merge logic for server deduplication and insertion
- Map initialization (new Map()) occurs at the start of configuration read-transform-write functions
- Server keying strategy is correctly applied: URL-based for SSE/SHTTP, name-based for STDIO
- Map-to-array conversion is performed before persistence operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep commands MUST execute successfully and return results matching the expected patterns before accepting configuration merge implementations.
</enforcement>