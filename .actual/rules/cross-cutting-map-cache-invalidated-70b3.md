# Use In-Memory Map for VSCode Server Configuration Caching: Map Cache Invalidated

These rules are ALWAYS ACTIVE for the VSCode settings module (src/vscode/settings.ts) and any code that manages server configuration state through in-memory caching.

### Rules

- **R-CACHE-001** MAY: The Map cache MAY be invalidated and rebuilt when filesystem changes are detected externally.
- **R-CACHE-002** MUST: Use `existingServerMap.set(server.name, server)` pattern consistently for all Map cache updates, with server.name as the key.
- **R-CACHE-003** MUST: Initialize existingServerMap as `new Map()` at module load time and populate from readVSCodeSettings on first access.
- **R-CACHE-004** SHOULD: Update Map cache first with set() operation in writeVSCodeSettings, then persist to filesystem to maintain consistency.
- **R-CACHE-005** MUST: Use server.name as Map key consistently across all set() and get() operations to ensure lookup correctness.
- **R-CACHE-006** SHOULD: Implement filesystem watch mechanism or cache invalidation strategy to handle external modifications; document that writeVSCodeSettings is the canonical write path.
- **R-CACHE-007** SHOULD: Monitor Map size in production; implement size limits or LRU eviction if configuration count exceeds memory-constrained thresholds.
- **R-CACHE-008** SHOULD: Implement write serialization through mutex or queue to prevent race conditions in concurrent writeVSCodeSettings calls.

### Verify

```bash
# Verify Map cache operations are used consistently
grep -n 'existingServerMap\.set' src/vscode/settings.ts

# Verify Map initialization
grep -n 'new Map()' src/vscode/settings.ts

# Verify JSON parsing from filesystem
grep -n 'JSON\.parse.*content' src/vscode/settings.ts
```

**Accept when:**
- Verification commands confirm existingServerMap.set() calls with server.name as key in src/vscode/settings.ts
- Map initialization is present and populated from JSON.parse operations on filesystem content
- Public API functions (readVSCodeSettings, writeVSCodeSettings) interact with Map cache structure
- Code review verification confirms server configuration access uses Map cache operations
- Unit tests validate Map.set() and Map.get() operations with server.name keys
- Static analysis detects no direct filesystem reads bypassing the cache layer for repeated configuration access

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-CACHE rules MUST be checked during code review. Violations require refactoring or architecture team approval for exceptions.
</enforcement>