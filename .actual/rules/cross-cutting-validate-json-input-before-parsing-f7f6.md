# Validate JSON Input Before Parsing in VSCode Settings Management UPDATE

These rules are ALWAYS ACTIVE for all files matching the configured scope, particularly src/vscode/settings.ts and any module that deserializes JSON input for VSCode settings or server configuration caching.

### Rules

- **R-JSON-001** MUST: Validate all parsed JSON objects against their expected schema (VSCodeSettings, AugmentMcpServer types) before inserting into the in-memory cache via existingServerMap.set() or equivalent persistence operations.
- **R-JSON-002** MUST: Catch and handle JSON.parse() errors explicitly, preventing malformed input from corrupting the cache layer or causing runtime failures.
- **R-JSON-003** SHOULD: Use schema validation libraries (e.g., zod, ajv) to enforce type safety and reject unexpected properties or missing required fields.

### Verify

```bash
# Check that JSON.parse() calls in src/vscode/settings.ts are wrapped in try-catch
grep -n "JSON\.parse" src/vscode/settings.ts | head -20

# Verify schema validation occurs before cache insertion
grep -A 5 "JSON\.parse" src/vscode/settings.ts | grep -E "(validate|schema|instanceof|type check)"

# Confirm existingServerMap.set() is only called after validation
grep -B 10 "existingServerMap\.set" src/vscode/settings.ts | grep -E "(validate|schema|check)"
```

**Accept when:**
- All JSON.parse() calls are wrapped in try-catch blocks
- Schema validation (type guards, schema validators, or instanceof checks) occurs before cache insertion
- Malformed JSON is logged or rejected without corrupting the in-memory cache
- VSCodeSettings and AugmentMcpServer types are enforced at the parsing boundary

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON deserialization in VSCode settings management MUST include explicit validation before cache operations.
</enforcement>