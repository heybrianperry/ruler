# Use Map-Based In-Memory Cache for MCP Server Registry State: Use Map Data

These rules are ALWAYS ACTIVE for all MCP server registry implementations in `src/mcp/` and VSCode settings management in `src/vscode/`, ensuring consistent in-memory caching of server configurations across transport types (SSE, SHTTP, STDIO).

### Rules

- **R-MAP-001** MUST: Use Map data structures to cache MCP server registry state indexed by server name or URL.
- **R-MAP-002** MUST: Maintain separate Map instances for each transport type (SSE, SHTTP, STDIO) to prevent key collisions between name-based and URL-based indexing schemes.
- **R-MAP-003** MUST: Use server.name as the key for STDIO servers and serverDef.url as the key for SSE/SHTTP servers.
- **R-MAP-004** MUST: Perform duplicate detection using Map.has() before inserting new entries via Map.set().
- **R-MAP-005** MUST: Initialize Map instances at module scope (existingSseServers, existingShttpServers, existingStdioServers) to maintain state across function calls.
- **R-MAP-006** SHOULD: Expose Map.size and Map.clear() through public API for observability and testing.
- **R-MAP-007** SHOULD: Implement file system watchers (fs.watch) to detect external configuration changes and trigger cache invalidation.

### Verify

```bash
# Verify Map instances are used for server registry caching
grep -r 'existingSseServers.set\|existingShttpServers.set\|existingStdioServers.set' src/

# Verify Map declarations exist for each transport type
grep -r 'new Map<' src/ | grep -E '(Sse|Shttp|Stdio)Server'

# Verify duplicate detection uses Map.has()
grep -r '\.has(' src/ | grep -E 'existing(Sse|Shttp|Stdio)Servers'
```

**Accept when:**
- All MCP server registry modules use Map.set() to store server entries indexed by name or URL
- Duplicate detection logic uses Map.has() before inserting new entries
- Separate Map instances exist for each transport type (SSE, SHTTP, STDIO) with appropriate key types
- Module-scope Map initialization is present in propagateOpenHandsMcp.ts and settings.ts
- STDIO servers use name-based keys and SSE/SHTTP servers use URL-based keys

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting changes to MCP server registry or VSCode settings modules.
</enforcement>