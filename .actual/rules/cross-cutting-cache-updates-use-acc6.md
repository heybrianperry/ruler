# Use In-Memory Map for VSCode Server Configuration Caching: Cache Updates Use

These rules are ALWAYS ACTIVE for the VSCode settings module (src/vscode/settings.ts) and any code that manages server configuration state requiring fast lookup and modification operations during runtime.

### Rules

- **R-CACHE-001** MUST: Cache updates MUST use Map.set() operations with server.name as the key and server object as the value

### Verify

```bash
# Verify Map.set() calls with server.name as key
grep -n 'existingServerMap\.set' src/vscode/settings.ts

# Verify Map initialization
grep -n 'new Map()' src/vscode/settings.ts

# Verify JSON.parse operations on filesystem content
grep -n 'JSON\.parse.*content' src/vscode/settings.ts
```

**Accept when:**
- Verification commands confirm existingServerMap.set() calls with server.name as key in src/vscode/settings.ts
- Map initialization is present and populated from JSON.parse operations on filesystem content
- Public API functions (readVSCodeSettings, writeVSCodeSettings) interact with Map cache structure
- All server configuration access uses Map cache operations rather than direct filesystem reads

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review rejection is required if configuration access bypasses Map cache without justification. Refactoring is required if direct filesystem I/O is used for repeated configuration reads.
</enforcement>