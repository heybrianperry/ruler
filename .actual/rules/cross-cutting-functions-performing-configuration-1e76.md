# Use Map-Based In-Memory Cache for Server Configuration State: Functions Performing Configuration

These rules are ALWAYS ACTIVE for configuration synchronization functions in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts that perform read-modify-write operations on server collections across multiple configuration file formats.

### Rules

- **R-CONFIG-001** SHOULD: Functions performing configuration synchronization SHOULD construct Maps from existing configuration arrays at the start of the operation and convert back to arrays before serialization.

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
- Map instances named existingServerMap, existingSseServers, existingShttpServers, or existingStdioServers are present in both src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts
- Map.set() operations are present in configuration synchronization functions for updating server cache entries
- Map instances are constructed within function scope rather than as module-level globals
- Array-to-Map conversion occurs at function entry and Map-to-array conversion occurs before serialization
- Appropriate keys are used per server type (server.name for STDIO, server.url for SSE/SHTTP)

<enforcement>
Claude Code MUST NOT skip or defer verification of Map-based caching patterns in configuration synchronization functions. All three verify commands MUST execute successfully before accepting changes to src/vscode/settings.ts or src/mcp/propagateOpenHandsMcp.ts.
</enforcement>