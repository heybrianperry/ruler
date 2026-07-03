# Use Map-Based In-Memory Caching for MCP Server Configuration State: Configuration Files Parsed

These rules are ALWAYS ACTIVE for all MCP server configuration management code paths in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts.

### Rules

- **R-CFG-001** MUST: Configuration files MUST be parsed and validated (via JSON.parse or parseTOML) before populating cache structures.
- **R-CFG-002** MUST: Initialize Map instances at function scope before parsing configuration files to ensure clean state for each operation.
- **R-CFG-003** MUST: Use consistent key naming: server.name for stdio servers, server.url or serverDef.url for SSE/HTTP servers.
- **R-CFG-004** MUST: Wrap JSON.parse() and parseTOML() calls in try-catch blocks with appropriate error handling and logging.
- **R-CFG-005** SHOULD: Consider extracting common cache-building logic into shared utility functions if pattern extends to additional configuration modules.
- **R-CFG-006** SHOULD: Document cache key selection rationale in code comments to prevent future inconsistencies.

### Verify

```bash
# Verify Map usage in configuration modules
grep -r 'Map<.*>' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify .set() operations on cache structures
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify parsing functions precede cache population
grep -r 'JSON\.parse\|parseTOML' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts
```

**Accept when:**
- All grep commands return matches confirming Map usage and .set() operations in both configuration modules
- Parsing functions (JSON.parse, parseTOML) are present before cache population logic
- No direct array-based or object-based caching patterns are found in the specified modules
- All parse operations are wrapped in try-catch blocks with error handling
- Cache key naming is consistent across both modules

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules during code review of configuration management changes.
</enforcement>