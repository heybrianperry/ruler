# Use Map-Based In-Memory Cache for Server Configuration State: Server Configuration State

These rules are ALWAYS ACTIVE for all files matching the configured scope: server configuration read-modify-write operations in `src/vscode/settings.ts` and `src/mcp/propagateOpenHandsMcp.ts`, transient state management during configuration file synchronization, deduplication logic for MCP server definitions across multiple configuration sources, and runtime operations involving frequent server lookups by name or URL.

### Rules

- **R-CACHE-001** MUST: Server configuration state MUST be cached in Map instances during read-modify-write operations, keyed by server name for STDIO servers and by URL for SSE/SHTTP servers.

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
- Grep commands identify Map instances named `existingServerMap`, `existingSseServers`, `existingShttpServers`, or `existingStdioServers` in both `src/vscode/settings.ts` and `src/mcp/propagateOpenHandsMcp.ts`
- Map.set() operations are present in configuration synchronization functions for updating server cache entries
- Map instances are constructed within function scope rather than as module-level globals
- Configuration synchronization functions use appropriate key selection (server.name for STDIO, server.url for SSE/SHTTP)
- Array-to-Map-to-array conversion logic is present for serialization to JSON or TOML formats

<enforcement>
Claude Code MUST NOT skip or defer verification. All three grep commands must succeed and identify Map-based caching patterns in the specified files before accepting configuration synchronization code.
</enforcement>