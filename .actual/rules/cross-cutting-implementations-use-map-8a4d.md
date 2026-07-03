# Adopt Map-Based In-Memory Cache for Server Configuration Management: Implementations Use Map

These rules are ALWAYS ACTIVE for all files matching the configured scope: functions performing server configuration read-transform-write cycles (readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands), server definition merging and deduplication logic across VSCode and OpenHands MCP integrations, and configuration modules handling SSE, SHTTP, and STDIO server types in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts.

### Rules

- **R-MAP-001** MAY: Implementations MAY use Map.has() for existence checks before conditional insertion to support merge semantics.
- **R-MAP-002** MUST: Use server.name as key for STDIO servers and server.url (or serverDef.url) as key for SSE/SHTTP servers to maintain consistent keying strategy.
- **R-MAP-003** MUST: Initialize Map instances at the start of configuration read operations and populate from parsed file content.
- **R-MAP-004** MUST: Convert Map back to array before serialization using Array.from(existingServerMap.values()) or [...existingServerMap.values()].
- **R-MAP-005** SHOULD: Scope Map variables with 'existing' prefix (existingServerMap, existingSseServers, existingShttpServers, existingStdioServers) to clearly indicate they represent current state from persistent storage.

### Verify

```bash
# Verify Map-based cache variables are present in configuration modules
grep -r 'existingServerMap\|existingSseServers\|existingShttpServers\|existingStdioServers' src/ --include='*.ts'

# Verify Map.set() operations are present in configuration merge logic
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts

# Verify Map initialization occurs at start of configuration functions
grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts
```

**Accept when:**
- Grep commands identify Map-based cache variables (existingServerMap, existingSseServers, existingShttpServers, existingStdioServers) in both src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts
- Map.set() operations are present in configuration merge logic for server deduplication and insertion
- Map initialization (new Map()) occurs at the start of configuration read-transform-write functions
- Server keying strategy is consistently applied: URL-based servers use URL as key, STDIO servers use name as key
- Map-to-array conversion is performed before persistence operations

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review verification that configuration merge functions use Map-based caching with appropriate keying strategies is mandatory. Static analysis or linting rules to detect array-based deduplication patterns (Array.find, Array.filter) in configuration modules must be applied. Unit tests validating O(1) lookup behavior and correct deduplication across server types are required.
</enforcement>