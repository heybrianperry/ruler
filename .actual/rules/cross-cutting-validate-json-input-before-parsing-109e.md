# Validate JSON Input Before Parsing in VSCode Settings Management UPDATE

These rules are ALWAYS ACTIVE for all files in `src/vscode/settings.ts` and any module that deserializes JSON from file system sources into in-memory caches or public contracts.

### Rules

- **R-JSON-001** MUST: Validate all JSON content read from the file system for well-formedness before calling `JSON.parse()`.
- **R-JSON-002** MUST: Wrap `JSON.parse()` calls in try-catch blocks to handle `SyntaxError` exceptions from malformed JSON.
- **R-JSON-003** MUST: Log or report validation failures with sufficient context (file path, error message) to aid debugging.
- **R-JSON-004** SHOULD: Use a schema validator (e.g., `ajv`, `zod`) to validate deserialized JSON against expected structure before persisting to cache.
- **R-JSON-005** SHOULD: Reject or sanitize any JSON that does not conform to the expected VSCode settings schema before calling `existingServerMap.set()`.

### Verify

```bash
# Check that JSON.parse() calls in src/vscode/settings.ts are preceded by validation or wrapped in try-catch
grep -n "JSON\.parse" src/vscode/settings.ts | head -20

# Verify that readVSCodeSettings and writeVSCodeSettings include error handling
grep -A 5 "readVSCodeSettings\|writeVSCodeSettings" src/vscode/settings.ts | grep -E "try|catch|validate"

# Confirm no direct JSON.parse() without surrounding error handling
grep -B 2 -A 2 "JSON\.parse" src/vscode/settings.ts | grep -v "try\|catch" || echo "All JSON.parse calls appear guarded"
```

**Accept when:**
- All `JSON.parse()` calls in the settings module are wrapped in try-catch blocks or preceded by explicit validation.
- Malformed JSON triggers an exception that is caught and logged, not silently ignored.
- The in-memory cache (`existingServerMap`) is never populated with invalid or partially-parsed data.
- Schema validation (if implemented) rejects JSON that does not match the expected VSCode settings structure.

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON deserialization boundaries in VSCode settings management MUST include validation before `JSON.parse()` is invoked.
</enforcement>