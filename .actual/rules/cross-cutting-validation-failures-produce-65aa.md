# Validate Parsed Configuration Data Using Type-Safe Predicates Before Access: Validation Failures Produce

These rules are ALWAYS ACTIVE for all configuration parsing and access patterns in test suites and application code that load external configuration files (TOML, JSON, YAML) from filesystem paths and environment variables.

### Rules

- **R-CONFIG-001** MUST: Validate parsed configuration data using type-safe predicates immediately after parsing (parseTOML(), JSON.parse(), or equivalent) and before any property access or array method calls.
- **R-CONFIG-002** MUST: Define TypeScript interfaces or Zod schemas for all configuration structures (mcp_servers, rulerDir configs, etc.) based on observed usage patterns.
- **R-CONFIG-003** MUST: Implement runtime type guards or schema validators that validate parsed 'any' types against defined interfaces before configuration data is accessed.
- **R-CONFIG-004** SHOULD: Validation failures SHOULD produce actionable error messages identifying the missing or malformed field and the expected schema.
- **R-CONFIG-005** MUST: Refactor .find(), .filter(), and .map() predicates to operate on validated, typed arrays rather than 'any[]' to enable type checking and autocomplete.
- **R-CONFIG-006** MUST: Add test cases that verify validation logic rejects malformed configuration and produces clear error messages identifying the schema violation.
- **R-CONFIG-007** MAY: Exempt test code using controlled fixtures with documented schemas from validation requirements (see exception EXC-001).

### Verify

```bash
# Count unvalidated parse calls lacking validation or schema guards
grep -r 'parseTOML\|JSON\.parse' --include='*.ts' | grep -v 'validate\|schema\|guard' | wc -l

# Count .find() predicates operating on 'any' typed arrays
grep -r '\.find((.*: any)' --include='*.ts' tests/ src/ | wc -l

# Verify configuration validation test cases exist and pass
npm test -- --testNamePattern='configuration.*validation' 2>&1 | grep -E 'PASS|FAIL'

# Verify TypeScript strict mode compilation with noImplicitAny
npx tsc --noImplicitAny --strict 2>&1 | grep -c 'error'
```

**Accept when:**
- All parseTOML() and JSON.parse() calls in src/ are followed by validation logic before configuration data is accessed.
- No .find() or .filter() predicates operate on arrays typed as 'any[]' in production code paths.
- Test suite includes validation test cases that verify rejection of malformed configuration with clear error messages.
- TypeScript strict mode compilation with noImplicitAny enabled produces zero errors for configuration parsing code.
- All configuration validation error messages identify the specific missing or malformed field and the expected schema.

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing code MUST pass validation checks before merge. Code review MUST block merge if configuration access patterns use 'any' types without validation. CI pipeline MUST fail if unvalidated parseTOML() or JSON.parse() calls are detected in new code.
</enforcement>