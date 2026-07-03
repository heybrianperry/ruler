# Validate JSON Input Before Parsing in VSCode Settings Management: Json Parse Operations

These rules are ALWAYS ACTIVE for all JSON.parse() operations on filesystem-sourced configuration data in VSCode settings management, particularly in src/vscode/settings.ts and related configuration deserialization paths.

### Rules

- **R-JSON-001** MUST: All JSON.parse() operations on filesystem-sourced configuration data MUST be preceded by input validation to verify content structure and type safety before deserialization.
- **R-JSON-002** MUST: JSON.parse() calls in readVSCodeSettings function MUST be wrapped in try-catch blocks with specific error handling for SyntaxError and validation failures.
- **R-JSON-003** MUST: Runtime validators for VSCodeSettings and AugmentMcpServer interfaces MUST be implemented using type guards or schema validation before deserialization.
- **R-JSON-004** MUST: Validation failures MUST be logged with sanitized error details to support troubleshooting without exposing sensitive configuration values.
- **R-JSON-005** SHOULD: FileSystemUtils SHOULD provide validated read operations as a reusable pattern across the codebase for configuration deserialization.

### Verify

```bash
# Check for unvalidated JSON.parse() in settings management
grep -n 'JSON\.parse' src/vscode/settings.ts | grep -v 'try\|catch\|validate'

# Verify all readVSCodeSettings usages include validation
grep -rn 'readVSCodeSettings' --include='*.ts' | xargs grep -L 'validate\|schema\|guard'

# Check for validation test coverage
npm test -- --grep 'malformed.*JSON|invalid.*settings' || echo 'No validation tests found'
```

**Accept when:**
- All JSON.parse() calls in src/vscode/settings.ts are wrapped in try-catch blocks with validation logic
- Test suite includes cases for malformed JSON, invalid schema, and missing required fields
- Grep commands return no matches indicating validation is present for all parse operations
- Runtime validators for VSCodeSettings and AugmentMcpServer are implemented and tested
- Validation failures are logged with appropriate error context

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse() operations on filesystem-sourced configuration data require explicit validation before deserialization. Violations must be caught during code review and CI pipeline checks.
</enforcement>