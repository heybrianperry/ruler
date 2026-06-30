# Use In-Memory Map for VSCode Server Configuration Caching: Server Configuration State

These rules are ALWAYS ACTIVE for the VSCode settings module (src/vscode/settings.ts) and all code that manages server configuration state requiring fast lookup and modification operations during runtime.

### Rules

- **R-VSCODE-CONFIG-001** MUST: Server configuration state MUST be cached in an in-memory Map structure keyed by server name.

### Verify

```bash
# Verify Map-based caching is used for server configuration
grep -n 'existingServerMap\.set' src/vscode/settings.ts
grep -n 'new Map()' src/vscode/settings.ts
grep -n 'JSON\.parse.*content' src/vscode/settings.ts
```

**Accept when:**
- Verification commands confirm existingServerMap.set() calls with server.name as key in src/vscode/settings.ts
- Map initialization is present and populated from JSON.parse operations on filesystem content
- Public API functions (readVSCodeSettings, writeVSCodeSettings) interact with Map cache structure
- Server configuration access uses Map cache operations (set/get) rather than direct filesystem reads

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration state access patterns must be reviewed to confirm Map-based caching is consistently applied. Code review rejection is required if configuration access bypasses the Map cache without documented justification.
</enforcement>