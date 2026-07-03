# Validate JSON Input Before Parsing in VSCode Settings Management: Validation Failures Logged

These rules are ALWAYS ACTIVE for all JSON deserialization operations on filesystem-sourced configuration data in VSCode settings management, particularly in `src/vscode/settings.ts` and related configuration handling modules.

### Rules

- **R-VSCODE-JSON-001** MUST: Wrap all JSON.parse() operations on filesystem-sourced configuration data in try-catch blocks with explicit error handling for SyntaxError and validation failures.
- **R-VSCODE-JSON-002** MUST: Implement schema validation before or immediately after JSON.parse() for data conforming to VSCodeSettings and AugmentMcpServer contracts.
- **R-VSCODE-JSON-003** SHOULD: Log validation failures with sufficient context to support debugging while avoiding exposure of sensitive configuration data in error messages.
- **R-VSCODE-JSON-004** MUST: Create and maintain runtime validators (type guards or schema validation) for all public API contracts used in configuration deserialization.
- **R-VSCODE-JSON-005** SHOULD: Implement validation as a reusable pattern in FileSystemUtils to enforce consistent validation across all filesystem-sourced deserialization operations.

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
- Runtime validators exist for VSCodeSettings and AugmentMcpServer interfaces
- Test suite includes cases for malformed JSON, invalid schema, and missing required fields
- Validation error messages are logged without exposing sensitive configuration values
- All grep commands return no matches indicating validation is present for all parse operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse() operations on filesystem-sourced configuration data MUST be validated before deserialization. Violations in security-sensitive modules require security team sign-off for exceptions.
</enforcement>