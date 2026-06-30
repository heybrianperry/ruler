# Use Map-Based Cache Layer for MCP Server Configuration State: Configuration Modules Use

These rules are ALWAYS ACTIVE for configuration modules managing MCP server definitions across multiple file formats, including VSCode settings (settings.ts) and OpenHands propagation (propagateOpenHandsMcp.ts).

### Rules

- **R-CONFIG-001** SHOULD: Configuration modules SHOULD use fs/promises or fs for file I/O operations to maintain consistency with the async/sync patterns in readVSCodeSettings and propagateMcpToOpenHands.
- **R-CONFIG-002** MUST: Use Map-based caching with appropriate keys (name for stdio servers, URL for SSE/SHTTP servers) for deduplication during server configuration merge operations.
- **R-CONFIG-003** MUST: Populate cache Map instances from existing configuration before performing merge operations to ensure deduplication works correctly.
- **R-CONFIG-004** MUST: Convert Map instances to arrays using Array.from(map.values()) before serializing to JSON or TOML formats.
- **R-CONFIG-005** SHOULD: Use TypeScript generics (Map<string, ServerType>) to maintain type safety across the cache layer and prevent type mismatches.
- **R-CONFIG-006** MUST: Implement key normalization functions that canonicalize names and URLs before Map.set operations to prevent key collisions.
- **R-CONFIG-007** MUST: Scope Map instances to function execution contexts (local variables) rather than module-level globals to prevent memory leaks.
- **R-CONFIG-008** SHOULD: When adding new server types, create a dedicated Map instance keyed by the appropriate identifier (name for stdio-like, URL for network-based).

### Verify

```bash
# Verify Map-based caching pattern usage
grep -r 'Map<string,' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() calls with appropriate keys
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|server\.url|newServer\.name|url|name)'

# Run deduplication and cache layer tests
npm test -- --grep 'deduplication|cache layer' 2>/dev/null || echo 'No specific cache tests found'

# Verify no duplicate server entries in output
grep -r 'duplicate' test/ 2>/dev/null || echo 'No duplicate detection tests found'
```

**Accept when:**
- All configuration modules use Map.set() with appropriate keys (name or URL) for caching server entries
- Grep commands confirm Map<string, ...> declarations and .set() calls in both settings.ts and propagateOpenHandsMcp.ts
- No duplicate server entries appear in output configuration files after merge operations
- Map instances are scoped to function execution contexts, not module-level globals
- Key normalization is applied before Map.set operations
- Array.from(map.values()) is used for JSON/TOML serialization

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST reject new configuration functions using array iteration for deduplication. CI pipeline MUST fail if integration tests detect duplicate server entries. Architecture review is required for any changes to the cache layer implementation pattern.
</enforcement>