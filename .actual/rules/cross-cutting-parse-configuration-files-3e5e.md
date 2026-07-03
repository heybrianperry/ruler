# Use Map-Based In-Memory Cache for MCP Server Registry State: Parse Configuration Files

These rules are ALWAYS ACTIVE for all MCP server registry implementations in `src/mcp/` and VSCode settings management in `src/vscode/`, particularly in files handling server configuration propagation and state management.

### Rules

- **R-CACHE-001** MUST: Parse configuration files (TOML via @iarna/toml, JSON via JSON.parse) before populating cache entries.
- **R-CACHE-002** MUST: Use separate Map instances for each transport type (SSE, SHTTP, STDIO) to prevent key collisions between name-based and URL-based indexing schemes.
- **R-CACHE-003** MUST: Use Map.set() to store server entries indexed by name (for STDIO) or URL (for SSE/SHTTP).
- **R-CACHE-004** MUST: Use Map.has() to check for duplicate entries before inserting new server entries.
- **R-CACHE-005** MUST: Initialize Map instances at module scope (existingSseServers, existingShttpServers, existingStdioServers) to maintain state across function calls.
- **R-CACHE-006** SHOULD: Expose Map.size and Map.clear() through public API for observability and testing.
- **R-CACHE-007** SHOULD: Implement periodic cache clearing or size limits to mitigate memory leak risks in long-running processes.
- **R-CACHE-008** SHOULD: Document single-threaded assumption and use async/await patterns to serialize file system operations.

### Verify

```bash
# Verify Map.set() usage for server entry storage
grep -r 'existingSseServers.set\|existingShttpServers.set\|existingStdioServers.set' src/

# Verify separate Map instances exist for each transport type
grep -r 'new Map<' src/ | grep -E '(Sse|Shttp|Stdio)Server'

# Verify Map.has() usage for duplicate detection
grep -r '\.has(' src/ | grep -E 'existing(Sse|Shttp|Stdio)Servers'

# Verify configuration file parsing before cache population
grep -r '@iarna/toml\|JSON.parse' src/mcp/ src/vscode/ | grep -B2 -A2 'existing.*Servers'
```

**Accept when:**
- All MCP server registry modules use Map.set() to store server entries indexed by name or URL
- Duplicate detection logic uses Map.has() before inserting new entries
- Separate Map instances exist for each transport type (SSE, SHTTP, STDIO) with appropriate key types
- Configuration files are parsed using @iarna/toml or JSON.parse before cache population
- Map instances are initialized at module scope to maintain state across function calls

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All MCP server registry implementations MUST comply with R-CACHE-001 through R-CACHE-005 (MUST-level rules) before code review approval.
</enforcement>