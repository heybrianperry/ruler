# Use Map-Based In-Memory Caching for MCP Server Configuration State: Cache Updates Use

These rules are ALWAYS ACTIVE for all MCP server configuration management code paths in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts, including functions readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands.

### Rules

- **R-CACHE-001** MUST: Cache updates MUST use Map.set() operations to ensure atomic key-value insertion or replacement.

### Verify

```bash
# Verify Map-based caching is used for server configuration
grep -r 'Map<.*>' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify .set() operations are used for cache updates
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
Claude Code MUST NOT skip or defer verification of Map.set() usage in configuration management code paths. All cache updates in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts MUST use Map.set() operations. Code review rejection is required for configuration code using non-Map caching approaches without documented exception approval.
</enforcement>