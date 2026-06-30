# Validate JSON Input Before Parsing in VSCode Settings Management: Parsed Configuration Objects

These rules are ALWAYS ACTIVE for all JSON deserialization operations on filesystem-sourced configuration data in VSCode settings management, particularly in `src/vscode/settings.ts` and related configuration handling modules.

### Rules

- **R-VSCODE-CONFIG-001** MUST: Parsed configuration objects MUST be validated against expected schemas (VSCodeSettings, AugmentMcpServer) before being stored in cache layers or used in application logic.

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
- Runtime validators exist for VSCodeSettings and AugmentMcpServer interfaces using type guards or schema validation
- Validation failures are logged with sanitized error details

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis tools MUST scan for unvalidated JSON.parse() patterns. Code review MUST enforce validation for all filesystem-sourced deserialization. CI pipeline MUST fail on detection of unvalidated JSON.parse() in configuration code paths.
</enforcement>