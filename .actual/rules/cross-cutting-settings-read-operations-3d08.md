# Validate JSON Input Before Parsing in VSCode Settings Management: Settings Read Operations

These rules are ALWAYS ACTIVE for all JSON deserialization operations on filesystem-sourced configuration data in VSCode settings management, particularly in `src/vscode/settings.ts` and related FileSystemUtils modules.

### Rules

- **R-SETTINGS-001** MUST: Settings read operations MUST implement error handling for JSON parsing failures with graceful degradation or safe defaults.
- **R-SETTINGS-002** MUST: All JSON.parse() calls in readVSCodeSettings function MUST be wrapped in try-catch blocks with specific error handling for SyntaxError and validation failures.
- **R-SETTINGS-003** MUST: Runtime validators for VSCodeSettings and AugmentMcpServer interfaces MUST be implemented using type guards or schema validation before data is stored in existingServerMap cache layer.
- **R-SETTINGS-004** MUST: Validation failures MUST be logged with sanitized error details to support troubleshooting without exposing sensitive configuration values.
- **R-SETTINGS-005** SHOULD: FileSystemUtils SHOULD provide validated read operations as a reusable pattern across the codebase for all filesystem-sourced deserialization.

### Verify

```bash
# Check for unvalidated JSON.parse() in settings module
grep -n 'JSON\.parse' src/vscode/settings.ts | grep -v 'try\|catch\|validate'

# Check for readVSCodeSettings usage without validation
grep -rn 'readVSCodeSettings' --include='*.ts' | xargs grep -L 'validate\|schema\|guard'

# Check for validation test coverage
npm test -- --grep 'malformed.*JSON|invalid.*settings' || echo 'No validation tests found'
```

**Accept when:**
- All JSON.parse() calls in src/vscode/settings.ts are wrapped in try-catch blocks with validation logic
- Test suite includes cases for malformed JSON, invalid schema, and missing required fields
- Grep commands return no matches indicating validation is present for all parse operations
- Runtime validators are implemented for VSCodeSettings and AugmentMcpServer contracts
- Validation failures are logged with appropriate error context

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis tools MUST scan for unvalidated JSON.parse() patterns. Code review MUST enforce validation for all filesystem-sourced deserialization. CI pipeline MUST fail on detection of unvalidated JSON.parse() in configuration code paths.
</enforcement>