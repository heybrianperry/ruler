# Validate JSON Input Before Parsing in VSCode Settings Management: Json Parse Operations

These rules are ALWAYS ACTIVE for all JSON parsing operations in VSCode settings management, particularly in the datastore layer (src/vscode/settings.ts) and any code that reads or writes configuration data from the filesystem.

### Rules

- **R-JSON-001** MUST: Wrap all JSON.parse() operations in try-catch blocks to handle parsing exceptions gracefully.
- **R-JSON-002** MUST: Implement type guard functions for VSCodeSettings and AugmentMcpServer that validate required fields and types before accepting parsed data.
- **R-JSON-003** MUST: Return default configuration on parse or validation failure in readVSCodeSettings operations.
- **R-JSON-004** MUST: Log validation failures with file path, error type, and sanitized partial content to aid debugging.
- **R-JSON-005** MUST: Validate data before serialization in writeVSCodeSettings to prevent writing invalid configuration.
- **R-JSON-006** SHOULD: Implement file integrity checks (checksums) for critical settings files to detect corruption early.
- **R-JSON-007** SHOULD: Cache validated results based on file modification time to mitigate performance degradation on large settings files.

### Verify

```bash
# Verify no unprotected JSON.parse calls exist in settings management
grep -n 'JSON\.parse' src/vscode/settings.ts | grep -v 'try\|catch'

# Verify validation functions are defined
grep -n 'function.*validate.*Settings\|isValid.*Settings' src/vscode/settings.ts

# Verify malformed JSON handling tests pass
npm test -- --grep 'settings.*validation.*malformed'
```

**Accept when:**
- All JSON.parse operations in src/vscode/settings.ts are wrapped in try-catch blocks
- Type guard functions exist for VSCodeSettings and AugmentMcpServer with required field validation
- Tests demonstrate graceful handling of malformed JSON with fallback to defaults and error logging
- Validation failures are logged with sufficient context for debugging
- No unprotected JSON.parse calls are detected by static analysis

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON parsing operations in scope MUST comply with R-JSON-001 through R-JSON-005 before code review approval. Violations detected in CI pipeline MUST block merge.
</enforcement>