# Adopt Map-Based In-Memory Cache for Server Configuration Management: Map Based Caches

These rules are ALWAYS ACTIVE for configuration management functions performing server read-transform-write cycles in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts, including readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands operations.

### Rules

- **R-MAP-001** SHOULD: Map-based caches SHOULD be scoped to function execution lifetime and not persisted beyond the configuration synchronization operation.
- **R-MAP-002** MUST: Use server.name as key for STDIO servers and server.url (or serverDef.url) as key for SSE/SHTTP servers to maintain consistent keying strategy.
- **R-MAP-003** MUST: Initialize Map instances at the start of configuration read operations with clear naming: const existingServerMap = new Map(); then populate from parsed file content.
- **R-MAP-004** MUST: Convert Map back to array before serialization using Array.from(existingServerMap.values()) or [...existingServerMap.values()].
- **R-MAP-005** SHOULD: Scope Map variables with 'existing' prefix (existingServerMap, existingSseServers, existingShttpServers, existingStdioServers) to clearly indicate they represent current state from persistent storage.
- **R-MAP-006** MUST NOT: Use array-based deduplication patterns (Array.find(), Array.filter()) for configuration merge operations in scope; use Map-based caching instead to achieve O(n) complexity.
- **R-MAP-007** MUST NOT: Persist Map instances beyond function execution lifetime; Maps are ephemeral intermediate state only.

### Verify

```bash
# Verify Map-based cache variables are present in configuration modules
grep -r 'existingServerMap\|existingSseServers\|existingShttpServers\|existingStdioServers' src/ --include='*.ts'

# Verify Map.set() operations in configuration merge logic
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts

# Verify Map initialization at function start
grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts

# Verify no array-based deduplication in scope
grep -r 'Array\.find\|Array\.filter' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts || echo "No array-based deduplication found (good)"
```

**Accept when:**
- Grep commands identify Map-based cache variables (existingServerMap, existingSseServers, existingShttpServers, existingStdioServers) in both src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts
- Map.set() operations are present in configuration merge logic for server deduplication and insertion
- Map initialization (new Map()) occurs at the start of configuration read-transform-write functions
- No array-based deduplication patterns (Array.find, Array.filter) are found in configuration merge operations
- Server keying strategy is consistent: STDIO servers use name as key, SSE/SHTTP servers use URL as key
- Map variables use 'existing' prefix to indicate they represent current state from persistent storage

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep commands MUST execute successfully and produce expected results before accepting configuration management code changes. Code review verification of Map-based caching with appropriate keying strategies is mandatory. Static analysis or linting rules detecting array-based deduplication patterns in configuration modules MUST be enforced. Unit tests validating O(1) lookup behavior and correct deduplication across server types MUST pass.
</enforcement>