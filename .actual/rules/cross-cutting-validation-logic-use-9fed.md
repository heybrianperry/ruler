# Validate JSON Input Before Parsing in VSCode Settings Management: Validation Logic Use

These rules are ALWAYS ACTIVE for all JSON parsing operations in VSCode settings management, particularly in `src/vscode/settings.ts` and any code that reads, writes, or caches VSCode configuration data.

### Rules

- **R-VSCODE-JSON-001** MUST: Wrap all `JSON.parse()` calls in try-catch blocks within `readVSCodeSettings` operations, returning default configuration on parse or validation failure.
- **R-VSCODE-JSON-002** MUST: Implement type guard functions for `VSCodeSettings` and `AugmentMcpServer` that validate required fields and their types before accepting parsed data.
- **R-VSCODE-JSON-003** MUST: Validate data before serialization in `writeVSCodeSettings` to prevent writing invalid configuration to the filesystem.
- **R-VSCODE-JSON-004** MUST: Log validation failures with file path, error type, and sanitized partial content to aid debugging and monitoring.
- **R-VSCODE-JSON-005** SHOULD: Implement file integrity checks (checksums) for critical settings files to detect corruption early.
- **R-VSCODE-JSON-006** MAY: Use schema validation libraries (e.g., Zod, Joi) for comprehensive type checking as schema complexity increases.

### Verify

```bash
# Verify no unprotected JSON.parse calls in settings management
grep -n 'JSON\.parse' src/vscode/settings.ts | grep -v 'try\|catch'

# Verify validation functions exist
grep -n 'function.*validate.*Settings\|isValid.*Settings' src/vscode/settings.ts

# Verify malformed JSON handling tests pass
npm test -- --grep 'settings.*validation.*malformed'
```

**Accept when:**
- All `JSON.parse` operations in `src/vscode/settings.ts` are wrapped in try-catch blocks
- Type guard functions exist for `VSCodeSettings` and `AugmentMcpServer` with required field validation
- Tests demonstrate graceful handling of malformed JSON with fallback to defaults and error logging
- Validation failures are logged with sufficient context for debugging
- Cache layer operations (`existingServerMap.set`) only receive validated data

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON parsing in settings management MUST be validated before use. Code review MUST block merges lacking validation logic for new parsing operations. CI pipeline MUST fail if unprotected JSON.parse is detected in the datastore layer.
</enforcement>