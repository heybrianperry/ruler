# Validate External Data with Parsing Functions Before Access Pattern Queries: Parsing Functions Invoked

These rules are ALWAYS ACTIVE for all configuration file loading and external data parsing operations in test suites and application code that reads TOML, JSON, or other structured data from external sources before applying data access patterns.

### Rules

- **R-PARSE-001** MUST: Parsing functions MUST be invoked immediately after reading file content and before any find(), filter(), or property access operations.

### Verify

```bash
# Detect parsing without access
grep -r 'parseTOML\|JSON\.parse' tests/ --include='*.ts' | grep -v 'find\|filter' || echo 'Parsing without access detected'

# Detect access without parsing
grep -B5 '\.find(' tests/ --include='*.ts' | grep -E 'parseTOML|JSON\.parse' || echo 'Access without parsing detected'

# Run configuration loading tests
npm test -- --testPathPattern='MistralVibeAgent|apply-engine' --verbose
```

**Accept when:**
- All configuration file reads are followed by parsing function calls before any data access operations
- Test suites for MistralVibeAgent and apply-engine successfully load and query configuration without runtime type errors
- Grep verification confirms parsing functions precede find/filter operations in configuration loading code paths
- No runtime type errors occur when accessing parsed configuration structures

<enforcement>
Clause R-PARSE-001 verification is mandatory. Code review MUST block merge if configuration loading bypasses parsing validation. CI pipeline MUST fail if tests demonstrate runtime errors from unvalidated configuration access. Architecture review is required for any new configuration loading patterns that deviate from parse-then-access flow.
</enforcement>