# Standardize Map-Based In-Memory Cache for Server Configuration State: Server Configuration Modules

These rules are ALWAYS ACTIVE for all server configuration modules that read from or write to VSCode settings.json, OpenHands config.toml, or expose public API functions for server registry operations (readVSCodeSettings, writeVSCodeSettings, propagateMcpToOpenHands).

### Rules

- **R-CACHE-001** SHOULD: Server configuration modules SHOULD separate concerns between file I/O (fs, fs/promises), parsing (JSON.parse, parseTOML), and cache management (Map operations).
- **R-CACHE-002** MUST: Use Map.set() for all server upsert operations to ensure idempotent semantics and avoid manual existence checks followed by insertion.
- **R-CACHE-003** MUST: Populate Map-based caches immediately after parsing configuration files (JSON.parse, parseTOML) and before any transformation or merge logic.
- **R-CACHE-004** SHOULD: Initialize separate Map instances for each server transport type (stdio, SSE, SHTTP) to enforce type-specific indexing strategies.
- **R-CACHE-005** MUST: When serializing Map-based caches back to configuration files, use Map.values() or Array.from(map.values()) to extract server objects, ensuring the output format matches the expected JSON/TOML schema.
- **R-CACHE-006** MUST: Add input validation in readVSCodeSettings and propagateMcpToOpenHands to verify that server objects contain required key fields (name, url) before Map.set() operations.
- **R-CACHE-007** SHOULD: Consider TypeScript type guards to enforce type safety between Map keys and server object properties.

### Verify

```bash
# Verify Map.set() operations using server.name or url as keys
grep -r 'Map.set' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | grep -E '(server\.name|url)' | wc -l

# Verify Map instantiations in server configuration modules
grep -r 'new Map<' src/vscode/settings.ts src/mcp/propagateOpenHandsMcp.ts | wc -l

# Verify unit tests for cache and Map behavior
npm test -- --testPathPattern='(settings|propagateOpenHandsMcp)' --testNamePattern='cache|Map'
```

**Accept when:**
- Grep commands confirm at least 4 Map.set() operations using server.name or url as keys across the two detected modules
- Grep commands confirm at least 2 Map instantiations in server configuration modules
- Unit tests for readVSCodeSettings, writeVSCodeSettings, and propagateMcpToOpenHands verify that Map-based caches correctly handle upsert, lookup, and serialization scenarios
- Input validation is present in readVSCodeSettings and propagateMcpToOpenHands to verify required server object fields before Map.set() operations

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review checklist items, static analysis via grep, and unit test coverage are mandatory before accepting server configuration module changes.
</enforcement>