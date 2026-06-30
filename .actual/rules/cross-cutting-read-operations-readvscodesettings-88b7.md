# Use In-Memory Map for VSCode Server Configuration Caching: Read Operations Readvscodesettings

These rules are ALWAYS ACTIVE for the VSCode settings module (src/vscode/settings.ts) and all code that manages server configuration state and caching operations.

### Rules

- **R-VSCODE-CACHE-001** SHOULD: Read operations (readVSCodeSettings) SHOULD access the Map cache rather than re-parsing filesystem data.

### Verify

```bash
# Verify Map-based caching is implemented
grep -n 'existingServerMap\.set' src/vscode/settings.ts
grep -n 'new Map()' src/vscode/settings.ts
grep -n 'JSON\.parse.*content' src/vscode/settings.ts
```

**Accept when:**
- Verification commands confirm existingServerMap.set() calls with server.name as key in src/vscode/settings.ts
- Map initialization is present and populated from JSON.parse operations on filesystem content
- Public API functions (readVSCodeSettings, writeVSCodeSettings) interact with Map cache structure
- Read operations use Map.get() or similar cache lookups instead of direct filesystem I/O

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All configuration access patterns must be reviewed to ensure Map cache is used for read operations.
</enforcement>