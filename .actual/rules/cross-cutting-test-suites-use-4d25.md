# Validate External Data with Parsing Functions Before Access Pattern Queries: Test Suites Use

These rules are ALWAYS ACTIVE for test suites loading configuration data from external sources (TOML files, JSON files) before applying data access patterns like find() queries.

### Rules

- **R-PARSE-001** MUST: Apply parsing functions (parseTOML, JSON.parse) immediately after reading external configuration files and before any data access operations (find, filter, property access).
- **R-PARSE-002** MUST: Wrap parsing operations in try-catch blocks to handle malformed files gracefully and provide clear error messages with file path and content context.
- **R-PARSE-003** SHOULD: Consider adding TypeScript type assertions or runtime schema validation after parsing to enforce expected structure before access patterns execute.
- **R-PARSE-004** MAY: Test suites MAY use this pattern to verify configuration loading behavior without requiring full application runtime.

### Verify

```bash
# Verify parsing functions are used before access operations
grep -r 'parseTOML\|JSON\.parse' tests/ --include='*.ts' | grep -v 'find\|filter' || echo 'Parsing without access detected'

# Verify access operations follow parsing in configuration loading
grep -B5 '\.find(' tests/ --include='*.ts' | grep -E 'parseTOML|JSON\.parse' || echo 'Access without parsing detected'

# Run test suites for MistralVibeAgent and apply-engine
npm test -- --testPathPattern='MistralVibeAgent|apply-engine' --verbose
```

**Accept when:**
- All configuration file reads are followed by parsing function calls before any data access operations
- Test suites for MistralVibeAgent and apply-engine successfully load and query configuration without runtime type errors
- Grep verification confirms parsing functions precede find/filter operations in configuration loading code paths
- Parsing operations are wrapped in try-catch blocks with contextual error handling

<enforcement>
Clause Code MUST NOT skip or defer verification. Code review MUST block merge if configuration loading bypasses parsing validation. CI pipeline MUST fail if tests demonstrate runtime errors from unvalidated configuration access. Architecture review is REQUIRED for any new configuration loading patterns that deviate from parse-then-access flow.
</enforcement>