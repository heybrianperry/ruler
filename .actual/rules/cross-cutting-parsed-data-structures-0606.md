# Validate External Data with Parsing Functions Before Access Pattern Queries: Parsed Data Structures

These rules are ALWAYS ACTIVE for all configuration file loading and external data parsing operations in test suites and application code that processes TOML, JSON, or other structured external data before applying data access patterns.

### Rules

- **R-PARSE-001** SHOULD: Parsed data structures SHOULD be type-checked or schema-validated to ensure expected properties exist before query operations (find, filter, property access).
- **R-PARSE-002** MUST: Parsing functions (parseTOML, JSON.parse) MUST be applied immediately after file content is read and before any data access patterns are executed.
- **R-PARSE-003** MUST: Parsing operations MUST be wrapped in try-catch blocks to handle malformed files gracefully and provide clear error messages with file context.
- **R-PARSE-004** SHOULD: Configuration loading code SHOULD maintain clear separation between validation (parsing) and querying (find/filter) stages for improved maintainability and testability.

### Verify

```bash
# Verify parsing functions are used before access patterns
grep -r 'parseTOML\|JSON\.parse' tests/ --include='*.ts' | grep -v 'find\|filter' || echo 'Parsing without access detected'

# Verify access patterns follow parsing operations
grep -B5 '\.find(' tests/ --include='*.ts' | grep -E 'parseTOML|JSON\.parse' || echo 'Access without parsing detected'

# Run configuration loading tests
npm test -- --testPathPattern='MistralVibeAgent|apply-engine' --verbose
```

**Accept when:**
- All configuration file reads are followed by parsing function calls before any data access operations
- Test suites for MistralVibeAgent and apply-engine successfully load and query configuration without runtime type errors
- Grep verification confirms parsing functions precede find/filter operations in configuration loading code paths
- Parsing operations include try-catch blocks with enriched error messages containing file path and context

<enforcement>
Clause Code MUST NOT skip or defer verification of parsing-before-access patterns. Code review MUST block merges if configuration loading bypasses parsing validation. CI pipeline MUST fail if tests demonstrate runtime errors from unvalidated configuration access. Architecture review is required for any new configuration loading patterns that deviate from parse-then-access flow.
</enforcement>