# Use Map-Based Cache Layer for MCP Server Configuration State: Public Contracts Vscodesettings

These rules are ALWAYS ACTIVE for all files matching the configured scope: MCP server configuration management in `src/vscode/settings.ts`, OpenHands MCP propagation in `src/mcp/propagateOpenHandsMcp.ts`, and operations involving `readVSCodeSettings`, `writeVSCodeSettings`, and `propagateMcpToOpenHands` functions.

### Rules

- **R-CACHE-001** SHOULD: Public API contracts (VSCodeSettings, AugmentMcpServer, readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands) SHOULD expose configuration operations without leaking cache implementation details.
- **R-CACHE-002** MUST: Use Map-based caching with natural identifiers (name for stdio servers, URL for SSE/SHTTP servers) as keys for deduplication during merge operations.
- **R-CACHE-003** MUST: Always populate the cache Map from existing configuration before performing merge operations to ensure deduplication works correctly.
- **R-CACHE-004** MUST: Convert Map instances to arrays using `Array.from(map.values())` before serializing to JSON or TOML formats.
- **R-CACHE-005** SHOULD: Use TypeScript generics (`Map<string, ServerType>`) to maintain type safety across the cache layer and prevent type mismatches.
- **R-CACHE-006** MUST: Implement key normalization functions that canonicalize names and URLs before `Map.set()` operations to prevent key collisions.
- **R-CACHE-007** MUST: Scope Map instances to function execution contexts (local variables) rather than module-level globals to prevent memory leaks.
- **R-CACHE-008** SHOULD: When adding new server types, create a dedicated Map instance keyed by the appropriate identifier (name for stdio-like, URL for network-based).
- **R-CACHE-009** MUST NOT: Use array-based configuration with filter/find operations for deduplication in configuration modules.
- **R-CACHE-010** MUST NOT: Use plain objects with bracket notation for caching (e.g., `cache[server.name] = server`) without proper type safety and prototype pollution guards.

### Verify

```bash
# Verify Map-based caching is used in configuration modules
grep -r 'Map<string,' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify .set() calls use appropriate keys
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|server\.url|newServer\.name|url|name)'

# Run cache and deduplication tests
npm test -- --grep 'deduplication|cache layer' 2>/dev/null || echo 'No specific cache tests found'

# Verify no duplicate server entries in output
grep -c 'name.*:' output-config.json | awk '{if ($1 > 1) print "FAIL: Duplicates detected"; else print "PASS: No duplicates"}'
```

**Accept when:**
- All configuration modules use `Map.set()` with appropriate keys (name or URL) for caching server entries
- Grep commands confirm `Map<string, ...>` declarations and `.set()` calls in both `settings.ts` and `propagateOpenHandsMcp.ts`
- No duplicate server entries appear in output configuration files after merge operations
- Key normalization functions are implemented and tested for common collision scenarios (trailing slashes, case sensitivity)
- Map instances are scoped to function execution contexts and properly cleared
- TypeScript generics are used consistently across cache layer declarations

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for configuration management code. Code review MUST reject any new configuration functions that use array iteration for deduplication instead of Map-based caching. CI pipeline MUST fail if integration tests detect duplicate server entries in output files.
</enforcement>