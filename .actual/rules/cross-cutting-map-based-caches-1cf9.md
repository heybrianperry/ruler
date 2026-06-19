# Standardize Map-Based In-Memory Cache for Server Configuration State: Map Based Caches

These rules are ALWAYS ACTIVE for all server configuration modules that read from or write to VSCode settings.json, OpenHands config.toml, or expose public API functions for server registry operations (readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands).

### Rules

- **R-MAP-001** MUST: Map-based caches MUST use server.name as the key for stdio server types and url as the key for SSE and SHTTP server types.
- **R-MAP-002** MUST: Use Map.set(key, value) for all server upsert operations to ensure idempotent semantics. Avoid manual existence checks followed by insertion.
- **R-MAP-003** MUST: Populate Map-based caches immediately after parsing configuration files (JSON.parse, parseTOML) and before any transformation or merge logic.
- **R-MAP-004** MUST: When serializing Map-based caches back to configuration files, use Map.values() or Array.from(map.values()) to extract server objects, ensuring the output format matches the expected JSON/TOML schema.
- **R-MAP-005** SHOULD: Initialize separate Map instances for each server transport type (stdio, SSE, SHTTP) to enforce type-specific indexing strategies.
- **R-MAP-006** SHOULD: Add input validation in readVSCodeSettings and propagateMcpToOpenHands to verify that server objects contain required key fields before Map.set() operations. Consider TypeScript type guards.

### Verify

```bash
# Verify Map.set() operations using server.name or url as keys
grep -r 'Map.set' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|url)' | wc -l

# Verify Map instantiations in server configuration modules
grep -r 'new Map<' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | wc -l

# Run unit tests for cache and Map behavior
npm test -- --testPathPattern='(settings|propagateOpenHandsMcp)' --testNamePattern='cache|Map'
```

**Accept when:**
- Grep commands confirm at least 4 Map.set() operations using server.name or url as keys across the two detected modules
- Grep commands confirm at least 2 Map instantiations in server configuration modules
- Unit tests for readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands verify that Map-based caches correctly handle upsert, lookup, and serialization scenarios

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review checklist items, static analysis via grep, and unit test coverage are mandatory before accepting changes to server configuration modules.
</enforcement>