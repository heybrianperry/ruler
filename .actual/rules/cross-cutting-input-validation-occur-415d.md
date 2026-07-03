# Use Map-Based Cache Layer for MCP Server Configuration State: Input Validation Occur

These rules are ALWAYS ACTIVE for all MCP server configuration management code, including VSCode settings operations, OpenHands MCP propagation, and any functions that read, transform, or write server definitions across multiple file formats.

### Rules

- **R-CACHE-001** MUST: Input validation MUST occur after parsing (JSON.parse, parseTOML) and before populating the cache layer.
- **R-CACHE-002** MUST: Use Map-based caching with O(1) lookup keyed by server name (for stdio servers) or URL (for SSE/SHTTP servers) for deduplication during merge operations.
- **R-CACHE-003** MUST: Normalize Map keys (canonicalize names and URLs, handle trailing slashes and case sensitivity) before Map.set operations to prevent key collisions.
- **R-CACHE-004** MUST: Scope Map instances to function execution contexts (local variables) rather than module-level globals to prevent memory leaks during configuration reloads.
- **R-CACHE-005** MUST: Convert Map instances to arrays using Array.from(map.values()) before serializing to JSON or TOML formats.
- **R-CACHE-006** SHOULD: Use TypeScript generics (Map<string, ServerType>) to maintain type safety across the cache layer and prevent type mismatches.
- **R-CACHE-007** SHOULD: Always populate the cache Map from existing configuration before performing merge operations to ensure deduplication works correctly.

### Verify

```bash
# Verify Map-based caching declarations in configuration modules
grep -r 'Map<string,' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() calls with appropriate keys
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|server\.url|newServer\.name|url|name)'

# Run deduplication and cache layer tests
npm test -- --grep 'deduplication|cache layer' 2>/dev/null || echo 'No specific cache tests found'

# Verify no duplicate server entries in output after merge
grep -r 'duplicate' test/ | grep -i 'server\|config' || echo 'No duplicate detection tests found'
```

**Accept when:**
- All configuration modules use Map.set() with appropriate keys (name or URL) for caching server entries
- Grep commands confirm Map<string, ...> declarations and .set() calls in both settings.ts and propagateOpenHandsMcp.ts
- No duplicate server entries appear in output configuration files after merge operations
- Input validation occurs immediately after JSON.parse or parseTOML and before any Map population
- Map instances are declared as local variables within function scopes, not at module level
- Array.from(map.values()) is used for serialization to JSON/TOML formats

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All configuration management code must pass the verify commands before acceptance. Violations require code review rejection and architecture review for exceptions.
</enforcement>