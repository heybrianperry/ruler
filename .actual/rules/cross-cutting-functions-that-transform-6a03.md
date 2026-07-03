# Use Map-Based Cache Layer for MCP Server Configuration State: Functions That Transform

These rules are ALWAYS ACTIVE for all files matching the configured scope: MCP server configuration management in src/vscode/settings.ts, OpenHands MCP propagation in src/mcp/propagateOpenHandsMcp.ts, and operations involving readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands functions.

### Rules

- **R-CACHE-001** MUST: Functions that transform configuration data MUST populate the cache layer before performing merge or update operations.
- **R-CACHE-002** MUST: Use Map<string, ServerType> with natural identifiers (name for stdio servers, URL for SSE/SHTTP servers) as keys for deduplication.
- **R-CACHE-003** MUST: Convert Map instances to arrays using Array.from(map.values()) before serializing to JSON or TOML formats.
- **R-CACHE-004** SHOULD: Implement key normalization functions that canonicalize names and URLs before Map.set operations to prevent key collisions.
- **R-CACHE-005** SHOULD: Scope Map instances to function execution contexts (local variables) rather than module-level globals to prevent memory leaks.
- **R-CACHE-006** SHOULD: Use TypeScript generics (Map<string, ServerType>) to maintain type safety across the cache layer.

### Verify

```bash
# Verify Map-based caching declarations in configuration modules
grep -r 'Map<string,' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() calls with appropriate keys
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|server\.url|newServer\.name|url|name)'

# Run deduplication and cache layer tests
npm test -- --grep 'deduplication|cache layer' 2>/dev/null || echo 'No specific cache tests found'

# Verify no duplicate server entries in output after merge
grep -c 'name.*:' output-config.json | awk '{if ($1 > 1) print "FAIL: Duplicates detected"; else print "PASS: No duplicates"}'
```

**Accept when:**
- All configuration modules use Map.set() with appropriate keys (name or URL) for caching server entries
- Grep commands confirm Map<string, ...> declarations and .set() calls in both settings.ts and propagateOpenHandsMcp.ts
- No duplicate server entries appear in output configuration files after merge operations
- Map instances are scoped to function execution contexts and properly cleared
- Key normalization is applied before Map.set operations for collision prevention

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review rejection is required if new configuration functions use array iteration for deduplication instead of Map-based caching. CI pipeline failure occurs if integration tests detect duplicate server entries in output files. Architecture review is required for any changes to the cache layer implementation pattern.
</enforcement>