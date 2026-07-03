# Use Map-Based In-Memory Cache for MCP Server Registry State: Use Server Name

These rules are ALWAYS ACTIVE for all MCP server registry implementations, VSCode settings management, and public API contracts that propagate server configurations to external systems.

### Rules

- **R-CACHE-001** MUST: Use server name as the Map key for STDIO transports and URL as the key for SSE/SHTTP transports.
- **R-CACHE-002** MUST: Initialize Map instances at module scope (existingSseServers, existingShttpServers, existingStdioServers) to maintain state across function calls.
- **R-CACHE-003** MUST: Use Map.set() for insertions and Map.has() for duplicate checks before adding new entries.
- **R-CACHE-004** SHOULD: Maintain separate Map instances per transport type to prevent key collisions between name-based (STDIO) and URL-based (SSE/SHTTP) indexing schemes.
- **R-CACHE-005** SHOULD: Parse configuration files using @iarna/toml for TOML and JSON.parse for JSON before populating cache.
- **R-CACHE-006** MAY: Expose Map.size and Map.clear() through public API for observability and testing.

### Verify

```bash
# Verify Map instances are initialized at module scope
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
- STDIO servers use server.name as the key; SSE/SHTTP servers use serverDef.url as the key
- Configuration files are parsed using @iarna/toml or JSON.parse before cache population

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-CACHE rules must be verified before accepting changes to MCP server registry implementations.
</enforcement>