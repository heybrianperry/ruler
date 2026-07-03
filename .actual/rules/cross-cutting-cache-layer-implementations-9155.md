# Use Map-Based Cache Layer for MCP Server Configuration State: Cache Layer Implementations

These rules are ALWAYS ACTIVE for all MCP server configuration management code, including VSCode settings operations and OpenHands MCP propagation functions that read, transform, and write server definitions across multiple file formats.

### Rules

- **R-CACHE-001** MUST: Cache layer implementations MUST use Map.set() to store entries with their natural identifier (name or URL) as the key.

### Verify

```bash
# Verify Map-based caching is used in configuration modules
grep -r 'Map<string,' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify .set() calls use appropriate keys
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|server\.url|newServer\.name|url|name)'

# Run deduplication and cache layer tests
npm test -- --grep 'deduplication|cache layer' 2>/dev/null || echo 'No specific cache tests found'
```

**Accept when:**
- All configuration modules use Map.set() with appropriate keys (name or URL) for caching server entries
- Grep commands confirm Map<string, ...> declarations and .set() calls in both settings.ts and propagateOpenHandsMcp.ts
- No duplicate server entries appear in output configuration files after merge operations
- Integration tests validate successful deduplication across configuration merge operations

<enforcement>
Clause R-CACHE-001 verification is mandatory. Code review MUST reject new configuration functions that use array iteration for deduplication instead of Map-based caching. CI pipeline MUST fail if integration tests detect duplicate server entries in output files.
</enforcement>