# Validate External Data with Parsing Functions Before Access Pattern Queries: External Data Loaded

These rules are ALWAYS ACTIVE for all code that loads external configuration data from files (TOML, JSON) before applying data access patterns like find() or filter() operations.

### Rules

- **R-EX-001** MUST: All external data loaded from files MUST be parsed through a validation function (parseTOML, JSON.parse, or equivalent) before applying data access patterns.
- **R-EX-002** MUST: Parsing operations MUST be wrapped in try-catch blocks to handle malformed files gracefully and provide clear error messages with file context.
- **R-EX-003** SHOULD: Consider adding TypeScript type assertions or runtime schema validation after parsing to enforce expected structure before access patterns.

### Verify

```bash
# Check for parsing functions used before access operations
grep -r 'parseTOML\|JSON\.parse' tests/ --include='*.ts' | grep -v 'find\|filter' || echo 'Parsing without access detected'

# Verify access operations are preceded by parsing
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
Clause Code MUST NOT skip or defer verification. Code review MUST block merge if configuration loading bypasses parsing validation. CI pipeline MUST fail if tests demonstrate runtime errors from unvalidated configuration access.
</enforcement>