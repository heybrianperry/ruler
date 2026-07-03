# Validate JSON Input Before Parsing in VSCode Settings Management: Settings Read Operations

These rules are ALWAYS ACTIVE for all settings read and write operations in the VSCode settings management module, particularly in `src/vscode/settings.ts` and any code that parses JSON configuration data for MCP server management.

### Rules

- **R-SETTINGS-001** MUST: Wrap all JSON.parse() calls in try-catch blocks within readVSCodeSettings operations and return default configuration on parse failure.
- **R-SETTINGS-002** MUST: Implement type guard functions for VSCodeSettings and AugmentMcpServer that validate required fields and data types before accepting parsed data.
- **R-SETTINGS-003** SHOULD: Settings read operations SHOULD provide fallback to default configuration when validation fails.
- **R-SETTINGS-004** MUST: Log validation failures with file path, error type, and sanitized partial content to aid debugging.
- **R-SETTINGS-005** MUST: Validate data before serialization in writeVSCodeSettings to prevent writing invalid configuration.
- **R-SETTINGS-006** SHOULD: Implement file integrity checks (checksums) for critical settings files to detect corruption early.
- **R-SETTINGS-007** MUST: Ensure cache layer operations (existingServerMap.set) only accept validated data to maintain consistency.

### Verify

```bash
# Verify no unprotected JSON.parse calls exist
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
- writeVSCodeSettings validates data before serialization
- Cache layer only receives validated data

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All JSON parsing operations in settings management must be protected and validated before use.
</enforcement>