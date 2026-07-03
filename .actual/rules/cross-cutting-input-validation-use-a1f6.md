# Validate JSON Input Before Parsing in VSCode Settings Management: Input Validation Use

These rules are ALWAYS ACTIVE for all JSON deserialization operations on filesystem-sourced configuration data in VSCode settings management, particularly in `src/vscode/settings.ts` and related configuration handling modules.

### Rules

- **R-INPUT-001** MUST: Validate all JSON input from filesystem sources before calling `JSON.parse()` using schema validation libraries or type guards to enforce contract compliance for public API contracts (VSCodeSettings, AugmentMcpServer).
- **R-INPUT-002** MUST: Wrap all `JSON.parse()` operations on filesystem-sourced data in try-catch blocks with specific error handling for `SyntaxError` and validation failures.
- **R-INPUT-003** SHOULD: Create and maintain runtime validators for VSCodeSettings and AugmentMcpServer interfaces using type guards or schema validation libraries compatible with existing TypeScript types.
- **R-INPUT-004** SHOULD: Log validation failures with sanitized error details to support troubleshooting without exposing sensitive configuration values.
- **R-INPUT-005** SHOULD: Update FileSystemUtils to provide validated read operations as a reusable pattern across the codebase for configuration deserialization.

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
- All `JSON.parse()` calls in `src/vscode/settings.ts` are wrapped in try-catch blocks with validation logic preceding deserialization
- Test suite includes cases for malformed JSON, invalid schema, and missing required fields
- Grep commands return no matches indicating validation is present for all parse operations on filesystem-sourced data
- Runtime validators exist for VSCodeSettings and AugmentMcpServer interfaces
- Validation failures are logged with appropriate error context

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse() operations on filesystem-sourced configuration data MUST be validated before deserialization. Violations in security-sensitive modules require security team sign-off for exceptions.
</enforcement>