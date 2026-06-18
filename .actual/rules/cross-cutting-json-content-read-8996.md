# Validate JSON Input Before Parsing in VSCode Settings Management: Json Content Read

These rules are ALWAYS ACTIVE for all JSON parsing operations in VSCode settings management, particularly in `src/vscode/settings.ts` and any module that reads or writes configuration data from the filesystem.

### Rules

- **R-JSON-001** MUST: All JSON content read from filesystem MUST be validated before parsing to ensure well-formed structure.
- **R-JSON-002** MUST: All readVSCodeSettings operations that parse JSON from filesystem MUST wrap JSON.parse() in try-catch blocks and return default configuration on parse or validation failure.
- **R-JSON-003** MUST: Type guard functions for VSCodeSettings and AugmentMcpServer MUST validate required fields and types before accepting parsed data into the cache layer (existingServerMap.set).
- **R-JSON-004** MUST: All writeVSCodeSettings operations MUST validate data before serialization to prevent writing invalid configuration.
- **R-JSON-005** SHOULD: Validation failures SHOULD be logged with file path, error type, and partial content (sanitized) to aid debugging.
- **R-JSON-006** MAY: File integrity checks (checksums) MAY be added for critical settings files to detect corruption early.

### Verify

```bash
# Verify no unprotected JSON.parse calls in settings management
grep -n 'JSON\.parse' src/vscode/settings.ts | grep -v 'try\|catch' # Should return no results

# Verify validation functions exist
grep -n 'function.*validate.*Settings\|isValid.*Settings' src/vscode/settings.ts # Should find validation functions

# Verify malformed JSON handling tests pass
npm test -- --grep 'settings.*validation.*malformed' # Should pass tests for malformed JSON handling
```

**Accept when:**
- All JSON.parse operations in src/vscode/settings.ts are wrapped in try-catch blocks
- Type guard functions exist for VSCodeSettings and AugmentMcpServer with required field validation
- Tests demonstrate graceful handling of malformed JSON with fallback to defaults and error logging
- Validation failures are logged with sufficient context for debugging
- No unprotected JSON.parse calls remain in the datastore layer

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline validation. Violations block merge and trigger runtime monitoring alerts.
</enforcement>