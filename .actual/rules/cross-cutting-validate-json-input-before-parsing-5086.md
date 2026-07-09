# Validate JSON Input Before Parsing in VSCode Settings Management UPDATE

These rules are ALWAYS ACTIVE for all files matching the configured scope, particularly src/vscode/settings.ts and related VSCode settings management modules that deserialize JSON input.

### Rules

- **R-VSCODE-JSON-001** MUST: Validate JSON input before calling JSON.parse() to prevent malformed data from corrupting the in-memory cache or causing runtime failures.
- **R-VSCODE-JSON-002** MUST: Implement try-catch blocks around JSON.parse() operations to gracefully handle parsing errors.
- **R-VSCODE-JSON-003** SHOULD: Trigger fallback to default configuration when invalid settings data is encountered, rather than allowing application failure.
- **R-VSCODE-JSON-004** MUST: Validate deserialized settings against expected schema (VSCodeSettings, AugmentMcpServer contracts) before persisting to existingServerMap cache.
- **R-VSCODE-JSON-005** SHOULD: Log validation failures and fallback events for debugging and monitoring purposes.

### Verify

```bash
# Check for JSON.parse() calls without validation or error handling
grep -n "JSON\.parse" src/vscode/settings.ts | grep -v "try\|catch" && echo "FAIL: Unprotected JSON.parse found" || echo "PASS: JSON.parse calls are protected"

# Verify fallback to default configuration exists
grep -n "default" src/vscode/settings.ts | grep -i "config\|settings" && echo "PASS: Default configuration fallback present" || echo "FAIL: No default configuration fallback"

# Check for schema validation before cache persistence
grep -n "existingServerMap\.set" src/vscode/settings.ts && grep -B5 "existingServerMap\.set" src/vscode/settings.ts | grep -i "validat" && echo "PASS: Validation before cache persistence" || echo "FAIL: Missing validation before cache persistence"
```

**Accept when:**
- All JSON.parse() calls are wrapped in try-catch blocks or preceded by validation logic
- Invalid settings data triggers fallback to default configuration instead of throwing uncaught errors
- Deserialized data is validated against VSCodeSettings and AugmentMcpServer contracts before being stored in existingServerMap
- Validation failures are logged for observability
- No malformed JSON can corrupt the in-memory cache layer

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON parsing boundaries in VSCode settings management must be validated before proceeding with cache operations.
</enforcement>