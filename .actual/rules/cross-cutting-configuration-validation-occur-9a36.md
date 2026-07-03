# Validate Parsed Configuration Data Using Type-Safe Predicates Before Access: Configuration Validation Occur

These rules are ALWAYS ACTIVE for all configuration parsing and access patterns in the codebase, including TOML, JSON, and YAML deserialization from external sources, environment variables, and filesystem paths.

### Rules

- **R-CFG-001** MUST: Configuration validation MUST occur immediately after parsing and before any business logic execution or property access.
- **R-CFG-002** MUST: All parseTOML(), JSON.parse(), and YAML deserialization calls in production code paths MUST be followed by explicit schema validation using type guards or Zod schemas before the parsed data is accessed.
- **R-CFG-003** MUST: Configuration structures accessed via .find(), .filter(), .map(), or property lookups MUST operate on validated, typed arrays rather than 'any[]' types.
- **R-CFG-004** SHOULD: Configuration validation logic SHOULD be co-located with parsing code and include clear error messages identifying schema violations.
- **R-CFG-005** SHOULD: TypeScript interfaces or Zod schemas SHOULD be defined for all configuration structures (mcp_servers, rulerDir configs, etc.) based on observed usage patterns.
- **R-CFG-006** MAY: Test code using controlled fixtures with documented schemas MAY be exempted from validation requirements (EXC-001), provided the schema guarantees are documented.

### Verify

```bash
# Count unvalidated parse calls lacking validation/schema/guard patterns
grep -r 'parseTOML\|JSON\.parse' --include='*.ts' src/ | grep -v 'validate\|schema\|guard' | wc -l

# Count .find() predicates operating on 'any' types in production code
grep -r '\.find((.*: any)' --include='*.ts' src/ | wc -l

# Verify configuration validation test coverage
npm test -- --testNamePattern='configuration.*validation' 2>&1 | grep -E 'PASS|FAIL'

# Verify TypeScript strict mode compilation
npx tsc --noImplicitAny --strict
```

**Accept when:**
- All parseTOML() and JSON.parse() calls in src/ are followed by validation logic before configuration data is accessed.
- No .find() or .filter() predicates operate on arrays typed as 'any[]' in production code paths.
- Test suite includes validation test cases that verify rejection of malformed configuration with clear error messages.
- TypeScript strict mode compilation with noImplicitAny enabled succeeds without errors.
- Code review checklist confirms validation logic for all new configuration parsing code.

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing must be validated before access. Violations detected in CI pipeline MUST block merge. Exceptions require tech lead approval and documented remediation plans.
</enforcement>