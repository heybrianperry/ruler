# Use Map-Based In-Memory Cache for MCP Server Registry State: Maintain Separate Map

These rules are ALWAYS ACTIVE for all MCP server registry operations in `src/mcp/propagateOpenHandsMcp.ts`, VSCode settings management in `src/vscode/settings.ts`, and public API contracts that manage server configurations across multiple transport types (SSE, SHTTP, STDIO).

### Rules

- **R-MAP-001** MUST: Maintain separate Map instances for each transport type (SSE, SHTTP, STDIO) to isolate lookup namespaces and prevent key collisions between name-based (STDIO) and URL-based (SSE/SHTTP) indexing schemes.
- **R-MAP-002** MUST: Initialize Map instances at module scope (existingSseServers, existingShttpServers, existingStdioServers) to maintain state across function calls.
- **R-MAP-003** MUST: Use Map.set() for insertions and Map.has() for duplicate checks before adding new entries to the registry.
- **R-MAP-004** MUST: Use server.name as the key for STDIO servers and serverDef.url as the key for SSE/SHTTP servers.
- **R-MAP-005** SHOULD: Expose Map.size and Map.clear() through public API for observability and testing purposes.
- **R-MAP-006** SHOULD: Implement periodic cache clearing or size limits to mitigate memory leak risks during long-running processes.
- **R-MAP-007** SHOULD: Document single-threaded assumption and use async/await patterns to serialize file system operations.

### Verify

```bash
# Verify Map.set() usage for all transport types
grep -r 'existingSseServers.set\|existingShttpServers.set\|existingStdioServers.set' src/

# Verify Map instances are declared with correct type signatures
grep -r 'new Map<' src/ | grep -E '(Sse|Shttp|Stdio)Server'

# Verify duplicate detection uses Map.has()
grep -r '\.has(' src/ | grep -E 'existing(Sse|Shttp|Stdio)Servers'
```

**Accept when:**
- All MCP server registry modules use Map.set() to store server entries indexed by name or URL
- Duplicate detection logic uses Map.has() before inserting new entries
- Separate Map instances exist for each transport type (SSE, SHTTP, STDIO) with appropriate key types
- STDIO servers are keyed by server.name and SSE/SHTTP servers are keyed by serverDef.url
- Map instances are initialized at module scope to maintain state across function calls

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. All pull requests modifying MCP server registry state must pass the verify commands and meet all accept criteria before merge.
</enforcement>