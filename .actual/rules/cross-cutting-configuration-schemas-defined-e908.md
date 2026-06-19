# Validate Parsed Configuration Data Using Type-Safe Predicates Before Access: Configuration Schemas Defined

These rules are ALWAYS ACTIVE for all configuration files loaded from filesystem paths, parsed from TOML/JSON/YAML sources, and accessed via array methods or property lookups in production code paths.

### Rules

- **R-CONFIG-001** MUST: Define TypeScript interfaces, Zod schemas, JSON Schema, or runtime type guards for all configuration structures before they are accessed via .find(), .filter(), .map(), or property lookups.
- **R-CONFIG-002** MUST: Validate parsed configuration data immediately after parseTOML(), JSON.parse(), or equivalent deserialization calls, before any property access or array method invocation.
- **R-CONFIG-003** MUST: Refactor .find() and .filter() predicates to operate on validated, typed arrays rather than 'any[]' to enable compile-time type checking and IDE autocomplete.
- **R-CONFIG-004** MUST: Produce clear, actionable error messages from validation logic that identify the specific schema violation and the configuration field that failed validation.
- **R-CONFIG-005** SHOULD: Co-locate schema definitions with configuration parsing code to maintain schema-implementation alignment and enable early detection of drift.
- **R-CONFIG-006** SHOULD: Include test cases that verify validation logic rejects malformed configuration and produces clear error messages identifying schema violations.
- **R-CONFIG-007** MAY: Implement validation in warn-only mode initially, monitor for validation failures in production, then enforce after validation schema is proven stable.

### Verify

```bash
# Count unvalidated parse calls lacking validation/schema/guard patterns
grep -r 'parseTOML\|JSON\.parse' --include='*.ts' src/ | grep -v 'validate\|schema\|guard' | wc -l

# Count .find() predicates operating on 'any' typed arrays in production code
grep -r '\.find((.*: any)' --include='*.ts' src/ | wc -l

# Verify configuration validation test coverage
npm test -- --testNamePattern='configuration.*validation' 2>&1 | grep -E 'PASS|FAIL'

# Verify TypeScript strict mode compilation
npx tsc --noImplicitAny --strict
```

**Accept when:**
- All parseTOML() and JSON.parse() calls in src/ are followed by validation logic before configuration data is accessed
- No .find() or .filter() predicates operate on arrays typed as 'any[]' in production code paths
- Test suite includes validation test cases that verify rejection of malformed configuration with clear error messages
- TypeScript compilation succeeds with noImplicitAny and strict mode enabled

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing code must be validated before merge. Test code using controlled fixtures with documented schemas (EXC-001) may be exempted with explicit documentation.
</enforcement>