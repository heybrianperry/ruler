# Use Map-Based In-Memory Cache for MCP Server Registry State: Check Map Cache

These rules are ALWAYS ACTIVE for all MCP server registry operations in src/mcp/propagateOpenHandsMcp.ts, VSCode settings management in src/vscode/settings.ts, and public API contracts that manage server configurations across multiple transport types (SSE, SHTTP, STDIO).

### Rules

- **R-CACHE-001** MUST: Check Map cache for existing entries before adding new server configurations to prevent duplicates.
- **R-CACHE-002** MUST: Use separate Map instances for each transport type (SSE, SHTTP, STDIO) with appropriate key types (server.name for STDIO, serverDef.url for SSE/SHTTP).
- **R-CACHE-003** MUST: Use Map.set() for insertions and Map.has() for duplicate checks before adding new entries.
- **R-CACHE-004** SHOULD: Initialize Map instances at module scope (existingSseServers, existingShttpServers, existingStdioServers) to maintain state across function calls.
- **R-CACHE-005** SHOULD: Expose Map.size and Map.clear() through public API for observability and testing.

### Verify

```bash
# Check for Map-based cache usage in MCP server registry modules
grep -r 'existingSseServers.set\|existingShttpServers.set\|existingStdioServers.set' src/

# Verify Map instances are declared for each transport type
grep -r 'new Map<' src/ | grep -E '(Sse|Shttp|Stdio)Server'

# Verify duplicate detection uses Map.has() before insertion
grep -r '\.has(' src/ | grep -E 'existing(Sse|Shttp|Stdio)Servers'
```

**Accept when:**
- All MCP server registry modules use Map.set() to store server entries indexed by name or URL
- Duplicate detection logic uses Map.has() before inserting new entries
- Separate Map instances exist for each transport type (SSE, SHTTP, STDIO) with appropriate key types
- Map instances are initialized at module scope to maintain state across function calls
- No alternative cache mechanisms (plain objects, external services) are used for server registry state

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All pull requests introducing or modifying MCP server registry operations MUST pass the verify commands and meet all accept criteria before approval.
</enforcement>