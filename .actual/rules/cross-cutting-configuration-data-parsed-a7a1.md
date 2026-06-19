# Validate Parsed Configuration Data Using Type-Safe Predicates Before Access: Configuration Data Parsed

These rules are ALWAYS ACTIVE for all configuration parsing and access patterns in the codebase, including test suites that load external configuration files (TOML, JSON, YAML) from filesystem paths and environment variables.

### Rules

- **R-CONFIG-001** MUST: All configuration data parsed from external sources (TOML, JSON, YAML) MUST be validated against an explicit schema or type guard before property access or predicate-based queries.
- **R-CONFIG-002** MUST: Configuration files loaded from filesystem paths (fs.readFile, fs/promises) MUST undergo schema validation immediately after parsing, before any .find(), .filter(), .map(), or property lookups.
- **R-CONFIG-003** MUST: Configuration data accessed using array methods (.find, .filter, .map) or property lookups MUST operate on validated, typed structures rather than 'any[]' types.
- **R-CONFIG-004** MUST: Environment variable-derived configuration paths (process.env.XDG_CONFIG_HOME) MUST be validated against schema before use in configuration loading logic.
- **R-CONFIG-005** SHOULD: Define TypeScript interfaces or Zod schemas for all configuration structures (mcp_servers, rulerDir configs) based on observed usage patterns and co-locate schema definitions with configuration parsing code.
- **R-CONFIG-006** SHOULD: Implement runtime type guards that validate parsed 'any' types immediately after parseTOML() or JSON.parse() calls and produce clear error messages identifying schema violations.
- **R-CONFIG-007** MAY: Test suites using controlled fixtures with guaranteed, documented schemas may be exempted from validation (EXC-001), provided the schema is explicitly documented and test coverage verifies the fixture structure.

### Verify

```bash
# Count unvalidated parse calls lacking validation/schema/guard patterns
grep -r 'parseTOML\|JSON\.parse' --include='*.ts' | grep -v 'validate\|schema\|guard' | wc -l

# Count .find() predicates operating on 'any' types
grep -r '\.find((.*: any)' --include='*.ts' tests/ src/ | wc -l

# Run configuration validation test suite
npm test -- --testNamePattern='configuration.*validation' 2>&1 | grep -E 'PASS|FAIL'
```

**Accept when:**
- All parseTOML() and JSON.parse() calls in src/ are followed by validation logic before configuration data is accessed
- No .find() or .filter() predicates operate on arrays typed as 'any[]' in production code paths
- Test suite includes validation test cases that verify rejection of malformed configuration with clear error messages
- Configuration validation logic is verified by TypeScript strict mode compilation with noImplicitAny enabled
- Code review checklist confirms validation logic for all new configuration parsing code
- CI pipeline successfully detects and blocks unvalidated parse calls in new code

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All configuration parsing and access patterns MUST be validated before merging. Violations detected by CI pipeline MUST block merge. Exception requests require tech lead approval and documented remediation plans.
</enforcement>