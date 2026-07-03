# Use Map-Based In-Memory Cache for Server Configuration State: Cache Updates Use

These rules are ALWAYS ACTIVE for all server configuration read-modify-write operations in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts, particularly during configuration file synchronization and deduplication logic for MCP server definitions.

### Rules

- **R-CACHE-001** MUST: Cache updates MUST use Map.set() operations to maintain O(1) insertion and lookup performance for server configuration entries.

### Verify

```bash
# Verify Map instances are used for server configuration caching
grep -r 'Map<.*>' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations are present in configuration synchronization
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map instances are constructed within function scope
grep -r 'new Map' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts
```

**Accept when:**
- Grep commands identify Map instances named existingServerMap, existingSseServers, existingShttpServers, or existingStdioServers in both src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts
- Map.set() operations are present in configuration synchronization functions for updating server cache entries
- Map instances are constructed within function scope rather than as module-level globals
- Configuration operations use appropriate keys (server.name for STDIO, server.url for SSE/SHTTP)
- Array-to-Map-to-array conversion logic is present for serialization to JSON or TOML formats

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based caching patterns in configuration synchronization functions. All three grep commands MUST pass before accepting changes to src/vscode/settings.ts or src/mcp/propagateOpenHandsMcp.ts that modify server configuration handling.
</enforcement>