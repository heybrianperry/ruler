# Use Map-Based In-Memory Cache for MCP Server Registry State: Validate Input Configuration

These rules are ALWAYS ACTIVE for all MCP server registry operations in `src/mcp/propagateOpenHandsMcp.ts`, VSCode settings management in `src/vscode/settings.ts`, and public API contracts that manage server configurations across multiple transport types (SSE, SHTTP, STDIO).

### Rules

- **R-CACHE-001** SHOULD: Validate input from configuration sources before inserting into Map cache to prevent malformed entries.
- **R-CACHE-002** MUST: Use separate Map instances for each transport type (SSE, SHTTP, STDIO) to prevent key collisions between name-based and URL-based indexing schemes.
- **R-CACHE-003** MUST: Use Map.set() for insertions and Map.has() for duplicate checks before adding new entries to the registry cache.
- **R-CACHE-004** MUST: Use server.name as the key for STDIO servers and serverDef.url as the key for SSE/SHTTP servers in their respective Map instances.
- **R-CACHE-005** SHOULD: Parse configuration files using @iarna/toml for TOML and JSON.parse for JSON before populating cache.
- **R-CACHE-006** SHOULD: Initialize Map instances at module scope (existingSseServers, existingShttpServers, existingStdioServers) to maintain state across function calls.
- **R-CACHE-007** SHOULD: Consider exposing Map.size and Map.clear() through public API for observability and testing.

### Verify

```bash
# Verify Map-based cache usage for server registry
grep -r 'existingSseServers.set\|existingShttpServers.set\|existingStdioServers.set' src/
grep -r 'new Map<' src/ | grep -E '(Sse|Shttp|Stdio)Server'
grep -r '\.has(' src/ | grep -E 'existing(Sse|Shttp|Stdio)Servers'
```

**Accept when:**
- All MCP server registry modules use Map.set() to store server entries indexed by name or URL
- Duplicate detection logic uses Map.has() before inserting new entries
- Separate Map instances exist for each transport type (SSE, SHTTP, STDIO) with appropriate key types
- Input validation occurs before cache insertion to prevent malformed entries
- Configuration parsing uses @iarna/toml and JSON.parse before populating cache

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All pull requests introducing or modifying MCP server registry state management MUST pass verification commands and meet acceptance criteria before merge.
</enforcement>