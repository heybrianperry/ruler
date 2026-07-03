# Use In-Memory Map for VSCode Server Configuration Caching: Write Operations Writevscodesettings

These rules are ALWAYS ACTIVE for the VSCode settings module (src/vscode/settings.ts) and all code that manages server configuration state through the public API contracts (readVSCodeSettings, writeVSCodeSettings, transformRulerToAugmentMcp).

### Rules

- **R-VSCODE-CACHE-001** SHOULD: Write operations (writeVSCodeSettings) SHOULD update both the Map cache and persist to filesystem.
- **R-VSCODE-CACHE-002** MUST: Use server.name as the Map key consistently across all set() and get() operations to ensure lookup correctness.
- **R-VSCODE-CACHE-003** MUST: Initialize existingServerMap as new Map() at module load time and populate from readVSCodeSettings on first access.
- **R-VSCODE-CACHE-004** SHOULD: In writeVSCodeSettings, update Map cache first with set() operation, then persist to filesystem to maintain consistency.
- **R-VSCODE-CACHE-005** MUST NOT: Bypass the Map cache layer for repeated configuration reads; all configuration access must use Map cache operations.

### Verify

```bash
# Verify Map-based caching is implemented with server.name as key
grep -n 'existingServerMap\.set' src/vscode/settings.ts

# Verify Map initialization at module load time
grep -n 'new Map()' src/vscode/settings.ts

# Verify filesystem content is parsed and used to populate cache
grep -n 'JSON\.parse.*content' src/vscode/settings.ts
```

**Accept when:**
- Verification commands confirm existingServerMap.set() calls with server.name as key in src/vscode/settings.ts
- Map initialization is present and populated from JSON.parse operations on filesystem content
- Public API functions (readVSCodeSettings, writeVSCodeSettings) interact with Map cache structure
- No direct filesystem reads bypass the cache layer for repeated configuration access
- Unit tests validate Map.set() and Map.get() operations with server.name keys

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review rejection is required if configuration access bypasses Map cache without justification. Refactoring is mandatory if direct filesystem I/O is used for repeated configuration reads. Architecture review is triggered if alternative caching mechanisms are proposed.
</enforcement>