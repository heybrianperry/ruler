# Use Map-Based In-Memory Cache for MCP Server Configuration State: Cache Instances Scoped

These rules are ALWAYS ACTIVE for MCP server configuration read/write operations in `src/vscode/settings.ts` and `src/mcp/propagateOpenHandsMcp.ts`, and for public API functions that expose configuration management: `readVSCodeSettings`, `writeVSCodeSettings`, `propagateMcpToOpenHands`, and `transformRulerToAugmentMcp`.

### Rules

- **R-CACHE-001** SHOULD: Cache instances SHOULD be scoped to the function execution context and not persisted beyond the configuration operation lifecycle.
- **R-CACHE-002** MUST: Use Map data structures for transient configuration state management to enable O(1) lookup and upsert operations.
- **R-CACHE-003** MUST: Key Map instances by server name (for STDIO transport) or URL (for SSE/SHTTP transport) to enable automatic deduplication during configuration merges.
- **R-CACHE-004** MUST: Initialize Map instances immediately after parsing configuration files (JSON.parse() or parseTOML()) and before any transformation logic.
- **R-CACHE-005** MUST: Use Map.set() for all upsert operations to leverage automatic deduplication; avoid manual existence checks before insertion.
- **R-CACHE-006** MUST: Convert Map back to array or object structure using Array.from(map.values()) or Object.fromEntries() before serialization to JSON/TOML.
- **R-CACHE-007** SHOULD: Use descriptive Map variable names that indicate transport type and purpose (e.g., existingSseServers, existingStdioServers, existingShttpServers).
- **R-CACHE-008** MUST NOT: Introduce persistent or application-wide cache instances for MCP server configuration state.
- **R-CACHE-009** MUST NOT: Extend Map-based caching pattern to longer-lived cache contexts without explicit design review and cache invalidation implementation.

### Verify

```bash
# Verify Map usage in configuration modules
grep -r 'new Map()' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations with appropriate key strategies
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|newServer\.name|url|serverDef\.url)'

# Verify Map usage in public API functions
grep -r 'readVSCodeSettings\|writeVSCodeSettings\|propagateMcpToOpenHands' src/ --include='*.ts' -A 20 | grep 'Map'
```

**Accept when:**
- All MCP server configuration read/write operations in `src/vscode/settings.ts` and `src/mcp/propagateOpenHandsMcp.ts` use Map-based caching with appropriate key strategies (name for STDIO, URL for SSE/SHTTP).
- Map.set() operations are present for upserting server entries with keys matching the transport type strategy.
- No persistent or application-wide cache instances are introduced for MCP server configuration state.
- Map instances are initialized immediately after parsing configuration files and before transformation logic.
- Map instances are converted back to array or object structures before serialization using Array.from(map.values()) or Object.fromEntries().
- Descriptive Map variable names indicating transport type and purpose are used throughout the codebase.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All configuration management code MUST follow the Map-based caching pattern with function-scoped lifecycle. Code review feedback MUST be provided for any deviations, and CI pipeline warnings MUST be generated for non-compliant configuration code.
</enforcement>