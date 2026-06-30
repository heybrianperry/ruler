# Use In-Memory Map for VSCode Server Configuration Caching: Cache Layer Populated

These rules are ALWAYS ACTIVE for the VSCode settings module (src/vscode/settings.ts) and any code that manages server configuration state and caching.

### Rules

- **R-CACHE-001** MUST: The cache layer MUST be populated from filesystem JSON data parsed via JSON.parse(content).
- **R-CACHE-002** MUST: Use Map data structure for O(1) lookup and update operations on server configurations keyed by server name.
- **R-CACHE-003** MUST: Initialize existingServerMap as new Map() at module load time and populate from readVSCodeSettings on first access.
- **R-CACHE-004** MUST: Use server.name as Map key consistently across all set() and get() operations to ensure lookup correctness.
- **R-CACHE-005** MUST: In writeVSCodeSettings, update Map cache first with set() operation, then persist to filesystem to maintain consistency.
- **R-CACHE-006** SHOULD: Implement filesystem watch mechanism or cache invalidation strategy to handle external process modifications.
- **R-CACHE-007** SHOULD: Document that writeVSCodeSettings is the canonical write path for configuration updates.
- **R-CACHE-008** SHOULD: Monitor Map size in production and implement size limits or LRU eviction if configuration count exceeds threshold.
- **R-CACHE-009** SHOULD: Implement write serialization through mutex or queue to prevent race conditions in concurrent writeVSCodeSettings calls.

### Verify

```bash
# Verify Map initialization and population
grep -n 'new Map()' src/vscode/settings.ts

# Verify Map.set() calls with server.name as key
grep -n 'existingServerMap\.set' src/vscode/settings.ts

# Verify JSON.parse operations on filesystem content
grep -n 'JSON\.parse.*content' src/vscode/settings.ts

# Verify public API functions interact with Map cache
grep -n 'readVSCodeSettings\|writeVSCodeSettings' src/vscode/settings.ts
```

**Accept when:**
- Verification commands confirm existingServerMap.set() calls with server.name as key in src/vscode/settings.ts
- Map initialization is present and populated from JSON.parse operations on filesystem content
- Public API functions (readVSCodeSettings, writeVSCodeSettings) interact with Map cache structure
- Server configuration access uses Map cache operations rather than direct filesystem reads
- Unit tests validate Map.set() and Map.get() operations with server.name keys

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-CACHE rules must be verified before accepting changes to src/vscode/settings.ts or related configuration caching logic.
</enforcement>