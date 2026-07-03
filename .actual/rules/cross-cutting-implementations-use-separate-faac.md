# Use Map-Based In-Memory Caching for MCP Server Configuration State: Implementations Use Separate

These rules are ALWAYS ACTIVE for all MCP server configuration management code paths in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts, including functions readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands.

### Rules

- **R-CACHE-001** MAY: Implementations MAY use separate Map instances for different server transport types (stdio, SSE, HTTP) to enforce type safety.
- **R-CACHE-002** MUST: Initialize Map instances at function scope before parsing configuration files to ensure clean state for each operation.
- **R-CACHE-003** MUST: Use consistent key naming: server.name for stdio servers, server.url or serverDef.url for SSE/HTTP servers.
- **R-CACHE-004** MUST: Wrap all JSON.parse() and parseTOML() calls in try-catch blocks with appropriate error handling and logging.
- **R-CACHE-005** MUST: Ensure Maps are function-scoped and not retained in closures to prevent memory leaks.
- **R-CACHE-006** SHOULD: Consider extracting common cache-building logic into shared utility functions if pattern extends to additional configuration modules.
- **R-CACHE-007** SHOULD: Document cache key selection rationale in code comments to prevent future inconsistencies.

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
- All JSON.parse() and parseTOML() calls are wrapped in try-catch blocks
- Map instances are declared at function scope, not in closures or module scope

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review checklist MUST require Map-based caching for new configuration management code. Static analysis rules MUST detect array.find() or object property access patterns in configuration modules. Integration tests MUST validate duplicate detection and merge behavior using Map semantics.
</enforcement>