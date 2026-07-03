# Use Map-Based In-Memory Caching for MCP Server Configuration State: Configuration Read Modify

These rules are ALWAYS ACTIVE for all MCP server configuration management code paths in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts, including functions readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands.

### Rules

- **R-CONFIG-001** SHOULD: Configuration read-modify-write operations SHOULD build complete cache state from existing configuration before applying modifications.

### Verify

```bash
# Verify Map-based caching structures are present in configuration modules
grep -r 'Map<.*>' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations are used for cache population
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify parsing functions precede cache population
grep -r 'JSON\.parse\|parseTOML' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts
```

**Accept when:**
- All grep commands return matches confirming Map usage and .set() operations in both configuration modules
- Parsing functions (JSON.parse, parseTOML) are present before cache population logic
- No direct array-based or object-based caching patterns are found in the specified modules
- Map instances are initialized at function scope before parsing configuration files
- Consistent key naming is used: server.name for stdio servers, server.url or serverDef.url for SSE/HTTP servers
- JSON.parse() and parseTOML() calls are wrapped in try-catch blocks with appropriate error handling

<enforcement>
Clause Code MUST NOT skip or defer verification of Map-based caching patterns in configuration management modules. All new configuration management code MUST use Map-based caching for server lookups and duplicate detection. Code review rejection is mandatory for configuration code using non-Map caching approaches without documented exception approval.
</enforcement>