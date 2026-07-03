# Use Map-Based In-Memory Caching for MCP Server Configuration State: Cache Key Selection

These rules are ALWAYS ACTIVE for all MCP server configuration management code paths in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts, including functions readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands.

### Rules

- **R-CACHE-001** SHOULD: Cache key selection SHOULD use stable identifiers (name for stdio, url for SSE/HTTP) that uniquely identify server instances.
- **R-CACHE-002** MUST: Initialize Map instances at function scope before parsing configuration files to ensure clean state for each operation.
- **R-CACHE-003** MUST: Use consistent key naming: server.name for stdio servers, server.url or serverDef.url for SSE/HTTP servers.
- **R-CACHE-004** MUST: Wrap all JSON.parse() and parseTOML() calls in try-catch blocks with appropriate error handling and logging.
- **R-CACHE-005** SHOULD: Ensure Maps are function-scoped and not retained in closures to prevent memory leaks.
- **R-CACHE-006** SHOULD: Consider extracting common cache-building logic into shared utility functions if pattern extends to additional configuration modules.

### Verify

```bash
# Verify Map usage in configuration modules
grep -r 'Map<.*>' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify .set() operations on cache Maps
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify parsing functions precede cache population
grep -r 'JSON\.parse\|parseTOML' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts
```

**Accept when:**
- All grep commands return matches confirming Map usage and .set() operations in both configuration modules
- Parsing functions (JSON.parse, parseTOML) are present before cache population logic
- No direct array-based or object-based caching patterns are found in the specified modules
- All parse operations are wrapped in try-catch blocks
- Map instances are declared at function scope, not in closures

<enforcement>
Clause Code MUST NOT skip or defer verification of Map-based caching patterns in configuration management modules. Violations require code review rejection or documented exception approval from the architecture review board.
</enforcement>