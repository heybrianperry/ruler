# Validate JSON Input Before Parsing in VSCode Settings Management: Parsed Json Objects

These rules are ALWAYS ACTIVE for all JSON parsing operations in VSCode settings management, particularly in `src/vscode/settings.ts` and related datastore layers that handle VSCodeSettings and AugmentMcpServer configuration objects.

### Rules

- **R-JSON-001** MUST: Parsed JSON objects MUST be validated against expected schema (VSCodeSettings, AugmentMcpServer types) before use in cache layers or public API contracts.
- **R-JSON-002** MUST: All JSON.parse() calls in readVSCodeSettings operations MUST be wrapped in try-catch blocks with graceful fallback to default configuration on parse or validation failure.
- **R-JSON-003** MUST: Type guard functions for VSCodeSettings and AugmentMcpServer MUST validate required fields and their types before accepting parsed data.
- **R-JSON-004** MUST: writeVSCodeSettings operations MUST validate data before serialization to prevent writing invalid configuration to the filesystem.
- **R-JSON-005** SHOULD: Validation failures SHOULD be logged with file path, error type, and sanitized partial content to aid debugging and monitoring.
- **R-JSON-006** SHOULD: Cache layer operations (existingServerMap.set) SHOULD only accept validated parsed configuration to maintain consistency and prevent cache corruption.

### Verify

```bash
# Check for unprotected JSON.parse calls in settings management
grep -n 'JSON\.parse' src/vscode/settings.ts | grep -v 'try\|catch'

# Verify validation functions exist for settings types
grep -n 'function.*validate.*Settings\|isValid.*Settings' src/vscode/settings.ts

# Run tests for malformed JSON handling
npm test -- --grep 'settings.*validation.*malformed'
```

**Accept when:**
- All JSON.parse operations in src/vscode/settings.ts are wrapped in try-catch blocks
- Type guard functions exist for VSCodeSettings and AugmentMcpServer with required field validation
- Tests demonstrate graceful handling of malformed JSON with fallback to defaults and error logging
- Validation failures are logged with sufficient context for debugging
- Cache layer only accepts validated parsed data

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All JSON parsing in settings management MUST include explicit validation before use. Code review and CI pipeline checks are mandatory before merge.
</enforcement>