# Use Map-Based In-Memory Cache for Server Configuration State: Implementation Use Separate

These rules are ALWAYS ACTIVE for configuration synchronization logic in src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts, particularly during read-modify-write cycles across multiple configuration file formats (JSON for VSCode, TOML for OpenHands).

### Rules

- **R-CACHE-001** MAY: Implementation MAY use separate Map instances for different server types (existingStdioServers, existingSseServers, existingShttpServers) to enforce type safety and key uniqueness constraints.
- **R-CACHE-002** MUST: Initialize Map instances at the start of configuration synchronization functions by iterating over existing server arrays and using appropriate keys (server.name for STDIO, server.url for SSE/SHTTP).
- **R-CACHE-003** MUST: Use Map.set() for all cache updates during the modification phase, ensuring idempotent behavior for duplicate server definitions.
- **R-CACHE-004** MUST: Convert Map instances back to arrays using Array.from(map.values()) before serializing to JSON or TOML formats.
- **R-CACHE-005** SHOULD: Extract Map construction and conversion logic into reusable utility functions to reduce code duplication across settings.ts and propagateOpenHandsMcp.ts.
- **R-CACHE-006** MUST NOT: Use Array.find() for server lookups in hot paths where Map-based O(1) lookups would be more efficient.
- **R-CACHE-007** MUST NOT: Declare Map instances as module-level globals; scope them to individual function invocations for transactional semantics.

### Verify

```bash
# Verify Map instances are present in configuration synchronization functions
grep -r 'Map<.*>' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map.set() operations are used for cache updates
grep -r '\.set(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(existingServerMap|existingSseServers|existingShttpServers|existingStdioServers)'

# Verify Map instances are constructed within function scope
grep -r 'new Map' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts

# Verify no Array.find() in hot paths for server lookups
grep -r 'Array\.find\|.find(' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -i server
```

**Accept when:**
- Grep commands identify Map instances named existingServerMap, existingSseServers, existingShttpServers, or existingStdioServers in both src/vscode/settings.ts and src/mcp/propagateOpenHandsMcp.ts
- Map.set() operations are present in configuration synchronization functions for updating server cache entries
- Map instances are constructed within function scope rather than as module-level globals
- Array.find() is not used in hot paths where Map lookups would provide O(1) performance
- Configuration operations complete within acceptable latency thresholds (< 100ms for conversion overhead)

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-CACHE rules MUST be verified before accepting pull requests that modify configuration synchronization logic. Violations MUST be flagged and require performance justification or technical lead approval for exceptions.
</enforcement>