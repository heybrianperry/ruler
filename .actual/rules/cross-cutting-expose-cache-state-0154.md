# Use Map-Based In-Memory Cache for MCP Server Registry State: Expose Cache State

These rules are ALWAYS ACTIVE for all MCP server registry implementations, VSCode settings management, and public API contracts that propagate server configurations to external systems.

### Rules

- **R-CACHE-001** MUST: Use Map-based in-memory cache for MCP server registry state indexed by name (STDIO) or URL (SSE/SHTTP).
- **R-CACHE-002** MUST: Maintain separate Map instances for each transport type (existingSseServers, existingShttpServers, existingStdioServers) to prevent key collisions.
- **R-CACHE-003** MUST: Use Map.has() to detect duplicate server entries before inserting new entries via Map.set().
- **R-CACHE-004** MUST: Use server.name as the key for STDIO servers and serverDef.url as the key for SSE/SHTTP servers.
- **R-CACHE-005** SHOULD: Expose Map.size and Map.clear() through public API contracts for observability and testing.
- **R-CACHE-006** MAY: Expose cache state through public API contracts (propagateMcpToOpenHands, readVSCodeSettings, writeVSCodeSettings, transformRulerToAugmentMcp) for external system integration.
- **R-CACHE-007** MUST NOT: Introduce persistent storage, database-backed caching, or external cache services for server registry state.
- **R-CACHE-008** MUST NOT: Use plain JavaScript objects with bracket notation for server registry cache storage.
- **R-CACHE-009** MUST NOT: Use WeakMap for server registry caching.

### Verify

```bash
# Verify Map instances are initialized at module scope
grep -r 'existingSseServers.set\|existingShttpServers.set\|existingStdioServers.set' src/

# Verify Map declarations exist for each transport type
grep -r 'new Map<' src/ | grep -E '(Sse|Shttp|Stdio)Server'

# Verify duplicate detection uses Map.has()
grep -r '\.has(' src/ | grep -E 'existing(Sse|Shttp|Stdio)Servers'

# Verify no plain object cache patterns
grep -r 'const.*=.*{}' src/mcp/ | grep -v 'Map' | grep -v '//' || echo 'No plain object caches detected'

# Verify no WeakMap usage for server registry
grep -r 'new WeakMap' src/ | grep -i server || echo 'No WeakMap server caches detected'
```

**Accept when:**
- All MCP server registry modules use Map.set() to store server entries indexed by name or URL
- Duplicate detection logic uses Map.has() before inserting new entries
- Separate Map instances exist for each transport type (SSE, SHTTP, STDIO) with appropriate key types
- No plain JavaScript objects are used as server registry cache storage
- No WeakMap instances are used for server registry state
- Public API contracts document cache state exposure and lifecycle

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules must be verified before accepting changes to MCP server registry implementations, VSCode settings management, or related public API contracts.
</enforcement>