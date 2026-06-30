# Validate JSON Input Before Parsing in VSCode Settings Management: Implementations Use Json

These rules are ALWAYS ACTIVE for all JSON deserialization operations on filesystem-sourced configuration data in VSCode settings management, particularly in `src/vscode/settings.ts` and related configuration handling modules.

### Rules

- **R-JSON-001** MUST: Wrap all `JSON.parse()` operations on filesystem-sourced data in try-catch blocks with explicit error handling for `SyntaxError` and validation failures.
- **R-JSON-002** MUST: Validate deserialized configuration data against expected schemas (VSCodeSettings, AugmentMcpServer) before storing in cache layers or using in application logic.
- **R-JSON-003** MAY: Implementations MAY use JSON schema validators, Zod, or TypeScript runtime type checkers to automate validation logic.
- **R-JSON-004** MUST: Create and maintain runtime validators (type guards or schema validators) for all public API contracts that receive filesystem-sourced JSON data.
- **R-JSON-005** MUST: Log validation failures with sanitized error details that support troubleshooting without exposing sensitive configuration values.
- **R-JSON-006** SHOULD: Implement comprehensive test coverage including malformed JSON, invalid schema, and missing required fields scenarios.

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
- All `JSON.parse()` calls in `src/vscode/settings.ts` are wrapped in try-catch blocks with validation logic
- Runtime validators exist for VSCodeSettings and AugmentMcpServer interfaces
- Test suite includes cases for malformed JSON, invalid schema, and missing required fields
- Grep commands return no matches indicating validation is present for all parse operations on filesystem-sourced data
- Validation failures are logged with appropriate error context

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse() operations on filesystem-sourced configuration data MUST include validation before deserialization. Violations in security-sensitive modules require security team sign-off for exceptions.
</enforcement>