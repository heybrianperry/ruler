# Validate JSON Input Before Parsing in VSCode Settings Management UPDATE

These rules are ALWAYS ACTIVE for all files in `src/vscode/settings.ts` and related modules that deserialize JSON input from file content into in-memory cache structures.

### Rules

- **R-JSON-001** MUST: Validate JSON input before calling `JSON.parse()` to prevent malformed data from corrupting the in-memory cache or causing runtime failures.
- **R-JSON-002** SHOULD: Log detailed error messages including file path and validation failure reason when validation fails.
- **R-JSON-003** MUST: Ensure all public contracts (`VSCodeSettings`, `AugmentMcpServer`, `readVSCodeSettings`, `writeVSCodeSettings`) implement input validation at the JSON parsing boundary.

### Verify

```bash
# Check that JSON.parse() calls in src/vscode/settings.ts are wrapped in try-catch or preceded by validation
grep -n "JSON\.parse" src/vscode/settings.ts | head -20

# Verify error handling logs file path and reason
grep -A 5 "catch.*Error" src/vscode/settings.ts | grep -E "(path|reason|message)"

# Confirm public contract functions validate input
grep -E "(readVSCodeSettings|writeVSCodeSettings|VSCodeSettings|AugmentMcpServer)" src/vscode/settings.ts | head -10
```

**Accept when:**
- All `JSON.parse()` calls are wrapped in try-catch blocks or guarded by schema validation
- Error handlers log the file path and specific validation failure reason
- Public contract functions validate input before passing to `JSON.parse()`
- No unvalidated JSON data is written to `existingServerMap` cache

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON input validation in VSCode settings management.
</enforcement>