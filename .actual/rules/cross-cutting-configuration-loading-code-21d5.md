# Validate External Data with Parsing Functions Before Access Pattern Queries: Configuration Loading Code

These rules are ALWAYS ACTIVE for configuration loading code in test suites and application initialization paths that read external TOML and JSON files before applying data access patterns.

### Rules

- **R-CFG-001** MUST: Separate parsing concerns from data access concerns by applying parsing functions (parseTOML, JSON.parse) immediately after file reads and before any find(), filter(), or property access operations on external configuration data.
- **R-CFG-002** MUST: Wrap all parsing operations in try-catch blocks to handle malformed files gracefully and provide clear error messages that include file path and content context.
- **R-CFG-003** SHOULD: Apply TypeScript type assertions or runtime schema validation immediately after parsing to enforce expected structure before access patterns execute.
- **R-CFG-004** SHOULD: Use well-maintained parsing libraries (@iarna/toml for TOML, native JSON.parse for JSON) and keep dependencies updated to mitigate parsing vulnerabilities.

### Verify

```bash
# Detect parsing without access (orphaned parsing)
grep -r 'parseTOML\|JSON\.parse' tests/ --include='*.ts' | grep -v 'find\|filter' || echo 'Parsing without access detected'

# Detect access without parsing (unvalidated access)
grep -B5 '\.find(' tests/ --include='*.ts' | grep -E 'parseTOML|JSON\.parse' || echo 'Access without parsing detected'

# Run configuration loading tests
npm test -- --testPathPattern='MistralVibeAgent|apply-engine' --verbose
```

**Accept when:**
- All configuration file reads are followed by parsing function calls before any data access operations
- Test suites for MistralVibeAgent and apply-engine successfully load and query configuration without runtime type errors
- Grep verification confirms parsing functions precede find/filter operations in configuration loading code paths
- All parsing operations include try-catch blocks with enriched error messages

<enforcement>
Claude Code MUST NOT skip or defer verification. Configuration loading code MUST apply parsing validation before access patterns. Code review MUST block merges that bypass parsing validation. CI pipeline MUST fail if tests demonstrate runtime errors from unvalidated configuration access.
</enforcement>