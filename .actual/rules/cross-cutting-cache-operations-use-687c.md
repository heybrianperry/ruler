# Adopt Map-Based In-Memory Cache for Server Configuration Management: Cache Operations Use

These rules are ALWAYS ACTIVE for all configuration management code performing server definition read-transform-write cycles, including VSCode settings integration and OpenHands MCP propagation modules.

### Rules

- **R-CACHE-001** MUST: Cache operations MUST use Map.set() for insertion/update to ensure O(1) lookup complexity and automatic key-based deduplication.
- **R-CACHE-002** MUST: Use server.name as key for STDIO servers and server.url (or serverDef.url) as key for SSE/SHTTP servers to maintain consistent keying strategy.
- **R-CACHE-003** MUST: Initialize Map instances at the start of configuration read operations with clear naming convention (existingServerMap, existingSseServers, existingShttpServers, existingStdioServers).
- **R-CACHE-004** MUST: Convert Map back to array before serialization using Array.from(map.values()) or [...map.values()] to prepare for persistence.
- **R-CACHE-005** SHOULD: Scope Map variables with 'existing' prefix to clearly indicate they represent current state from persistent storage.
- **R-CACHE-006** MUST NOT: Use array-based deduplication patterns (Array.find(), Array.filter()) for configuration merge operations in scope.

### Verify

```bash
# Verify Map-based cache variables are present in configuration modules
grep -r 'existingServerMap\|existingSseServers\|existingShttpServers\|existingStdioServers' src/ --include='*.ts'

# Verify Map.set() operations are used in configuration merge logic
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts

# Verify Map initialization occurs at function start
grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts

# Verify no array-based deduplication patterns in scope
grep -r 'Array\.find\|Array\.filter' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts || echo 'No array-based patterns found (good)'
```

**Accept when:**
- Grep commands identify Map-based cache variables (existingServerMap, existingSseServers, etc.) in both src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts
- Map.set() operations are present in configuration merge logic for server deduplication and insertion
- Map initialization (new Map()) occurs at the start of configuration read-transform-write functions
- No array-based deduplication patterns (Array.find, Array.filter) are found in configuration modules
- Map-to-array conversion using Array.from() or spread operator occurs before persistence operations
- Keying strategy is correctly applied: URL-based keys for SSE/SHTTP servers, name-based keys for STDIO servers

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All configuration merge functions in scope MUST use Map-based caching with appropriate keying strategies. Violations require code review feedback and refactoring before acceptance.
</enforcement>