# Use Map-Based Cache Layer for MCP Server Configuration State: Configuration Read Write

These rules are ALWAYS ACTIVE for all configuration read/write operations in MCP server management, specifically in `src/vscode/settings.ts` and `src/mcp/propagateOpenHandsMcp.ts`, and any new configuration modules that handle server collections keyed by name (stdio) or URL (SSE/SHTTP).

### Rules

- **R-CONFIG-001** MUST: Configuration read/write operations MUST use Map data structures keyed by server name (for stdio) or URL (for SSE/SHTTP) to cache and deduplicate server entries.
- **R-CONFIG-002** MUST: Always populate the cache Map from existing configuration before performing merge operations to ensure deduplication works correctly.
- **R-CONFIG-003** MUST: Convert Map instances to arrays using `Array.from(map.values())` before serializing to JSON or TOML formats.
- **R-CONFIG-004** MUST: Use TypeScript generics (`Map<string, ServerType>`) to maintain type safety across the cache layer and prevent type mismatches.
- **R-CONFIG-005** MUST: Implement key normalization functions that canonicalize names and URLs before `Map.set()` operations to prevent key collisions.
- **R-CONFIG-006** MUST: Ensure Map instances are scoped to function execution contexts (local variables) rather than module-level globals to prevent memory leaks.
- **R-CONFIG-007** SHOULD: When adding new server types, create a dedicated Map instance keyed by the appropriate identifier (name for stdio-like, URL for network-based).
- **R-CONFIG-008** SHOULD: Add validation tests for common collision scenarios (trailing slashes, case sensitivity) in key normalization.

### Verify

```bash
# Verify Map-based caching is used in configuration modules
grep -r 'Map<string,' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify .set() calls use appropriate keys
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|server\.url|newServer\.name|url|name)'

# Run deduplication and cache layer tests
npm test -- --grep 'deduplication|cache layer' 2>/dev/null || echo 'No specific cache tests found'

# Verify no duplicate server entries in output
grep -c 'name.*:' output-config.json | awk '{if ($1 > 1) print "FAIL: Duplicates detected"; else print "PASS: No duplicates"}'
```

**Accept when:**
- All configuration modules use `Map.set()` with appropriate keys (name or URL) for caching server entries.
- Grep commands confirm `Map<string, ...>` declarations and `.set()` calls in both `settings.ts` and `propagateOpenHandsMcp.ts`.
- No duplicate server entries appear in output configuration files after merge operations.
- Key normalization functions are implemented and tested for common collision scenarios.
- Map instances are scoped to function execution contexts, not module-level globals.
- Integration tests validate no duplicate servers after configuration merge operations.

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration read/write operations MUST be reviewed against these rules before merge. Code review rejection is mandatory if new configuration functions use array iteration for deduplication instead of Map-based caching. CI pipeline failure is required if integration tests detect duplicate server entries in output files.
</enforcement>