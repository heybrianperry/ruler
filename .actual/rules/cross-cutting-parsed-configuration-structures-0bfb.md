# Validate Parsed Configuration Data Using Type-Safe Predicates Before Access: Parsed Configuration Structures

These rules are ALWAYS ACTIVE for all configuration parsing and access patterns in the codebase, including TOML, JSON, and YAML deserialization from external sources, environment variables, and filesystem paths.

### Rules

- **R-CONFIG-001** MUST: Parsed configuration structures typed as 'any' MUST NOT be directly accessed using .find(), .filter(), or property lookups without prior type narrowing or validation.
- **R-CONFIG-002** MUST: All parseTOML() and JSON.parse() calls MUST be immediately followed by validation logic before configuration data is accessed in production code paths.
- **R-CONFIG-003** MUST: Type guards or Zod schemas MUST validate parsed 'any' types against defined TypeScript interfaces immediately after deserialization.
- **R-CONFIG-004** SHOULD: Configuration structures MUST be refactored to operate on validated, typed arrays rather than 'any[]' to enable type checking and IDE autocomplete.
- **R-CONFIG-005** SHOULD: Validation logic SHOULD produce clear, actionable error messages identifying the specific schema violation when rejecting malformed configuration.

### Verify

```bash
# Count unvalidated parse calls lacking validation/schema/guard patterns
grep -r 'parseTOML\|JSON\.parse' --include='*.ts' src/ | grep -v 'validate\|schema\|guard' | wc -l

# Count .find() predicates operating on 'any' typed arrays in production code
grep -r '\.find((.*: any)' --include='*.ts' src/ | wc -l

# Verify configuration validation test coverage
npm test -- --testNamePattern='configuration.*validation' 2>&1 | grep -E 'PASS|FAIL'
```

**Accept when:**
- All parseTOML() and JSON.parse() calls in src/ are followed by validation logic before configuration data is accessed
- No .find() or .filter() predicates operate on arrays typed as 'any[]' in production code paths
- Test suite includes validation test cases that verify rejection of malformed configuration with clear error messages
- TypeScript strict mode compilation with noImplicitAny enabled passes without errors
- Code review checklist confirms validation logic for all new configuration parsing code

<enforcement>
Clause MUST NOT skip or defer verification. All configuration parsing code must be validated before merge. Test code using controlled fixtures with documented schemas may be exempted via exception process requiring tech lead approval and remediation plan.
</enforcement>