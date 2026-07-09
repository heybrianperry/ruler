# Validate JSON Input Before Parsing in VSCode Settings Management UPDATE

These rules are ALWAYS ACTIVE for all files matching the configured scope, particularly src/vscode/settings.ts and related modules that deserialize JSON input from VSCode settings files.

### Rules

- **R-JSON-001** MUST: Validate JSON input at the parsing boundary before calling JSON.parse() to prevent malformed data from corrupting the in-memory cache or causing runtime failures.
- **R-JSON-002** SHOULD: Use schema validation libraries (e.g., Zod, Joi) for type-safe validation of deserialized VSCode settings data.
- **R-JSON-003** MUST: Ensure all public contracts (VSCodeSettings, AugmentMcpServer, readVSCodeSettings, writeVSCodeSettings) validate input before persisting to existingServerMap cache layer.

### Verify

```bash
# Check for JSON.parse() calls without preceding validation
grep -n "JSON\.parse" src/vscode/settings.ts | grep -v "validate\|schema\|Zod\|Joi" || echo "All JSON.parse calls appear to have validation context"

# Verify schema validation is applied to VSCode settings deserialization
grep -A 5 "readVSCodeSettings\|JSON\.parse" src/vscode/settings.ts | grep -E "validate|schema|Zod|Joi" && echo "Schema validation detected"

# Check existingServerMap.set() operations are guarded by validation
grep -B 5 "existingServerMap\.set" src/vscode/settings.ts | grep -E "validate|schema|try.*catch" && echo "Cache operations are guarded"
```

**Accept when:**
- All JSON.parse() calls in settings.ts are preceded by input validation logic
- Schema validation library (Zod, Joi, or equivalent) is applied to VSCode settings objects
- existingServerMap.set() operations only execute after successful validation
- Public contracts (readVSCodeSettings, writeVSCodeSettings) validate input before file system or cache operations
- Malformed JSON input is caught and handled gracefully without corrupting the cache

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON input validation at the parsing boundary. All modifications to src/vscode/settings.ts must maintain or strengthen validation coverage.
</enforcement>