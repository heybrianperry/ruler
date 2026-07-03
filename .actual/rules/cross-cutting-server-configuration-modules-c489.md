# Adopt Map-Based In-Memory Cache for Server Configuration Management: Server Configuration Modules

These rules are ALWAYS ACTIVE for server configuration modules performing read-transform-write cycles across VSCode settings and OpenHands MCP integrations, specifically in `src/vscode/settings.ts` and `src/mcp/propagateOpenHandsMcp.ts`.

### Rules

- **R-CACHE-001** MUST: Server configuration modules MUST use Map data structures with appropriate keys (URL for SSE/SHTTP servers, name for STDIO servers) as the intermediate cache layer during configuration read-transform-write operations.
- **R-CACHE-002** MUST: Initialize Map instances at the start of configuration read operations with clear naming convention using 'existing' prefix (existingServerMap, existingSseServers, existingShttpServers, existingStdioServers).
- **R-CACHE-003** MUST: Use server.name as key for STDIO servers and server.url (or serverDef.url) as key for SSE/SHTTP servers to maintain consistent keying strategy across server types.
- **R-CACHE-004** MUST: Convert Map back to array before serialization using Array.from(map.values()) or [...map.values()] to prepare for persistence operations.
- **R-CACHE-005** SHOULD: Document keying strategy in type definitions and function documentation; consider TypeScript discriminated unions to enforce correct key usage at compile time.
- **R-CACHE-006** SHOULD: Explicitly document that server ordering is not guaranteed after Map-to-array conversion; implement explicit sorting if ordering is required.

### Verify

```bash
# Verify Map-based cache variables are present in configuration modules
grep -r 'existingServerMap\|existingSseServers\|existingShttpServers\|existingStdioServers' src/ --include='*.ts'

# Verify Map.set() operations in configuration merge logic
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts

# Verify Map initialization at function start
grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts
```

**Accept when:**
- Grep commands identify Map-based cache variables (existingServerMap, existingSseServers, existingShttpServers, existingStdioServers) in both src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts
- Map.set() operations are present in configuration merge logic for server deduplication and insertion
- Map initialization (new Map()) occurs at the start of configuration read-transform-write functions
- No array-based deduplication patterns (Array.find, Array.filter) are found in configuration merge logic
- Keying strategy is consistently applied: URL-based keys for SSE/SHTTP servers, name-based keys for STDIO servers

<enforcement>
Claude Code MUST NOT skip or defer verification. All grep commands MUST execute successfully and return results matching the accept criteria before approving configuration module changes.
</enforcement>