# Validate External Data with Parsing Functions Before Access Pattern Queries: Data Access Patterns

These rules are ALWAYS ACTIVE for all configuration loading and external data access patterns in test suites and runtime code that processes TOML, JSON, or other structured external data sources.

### Rules

- **R-DAP-001** MUST: Data access patterns (find, filter) applied to parsed external data MUST handle cases where expected entries are not found.
- **R-DAP-002** MUST: Parse external configuration data using dedicated parsing functions (parseTOML, JSON.parse) immediately after file read operations and before any data access patterns are applied.
- **R-DAP-003** MUST: Wrap parsing operations in try-catch blocks to handle malformed files gracefully and provide clear error messages with file path and content context.
- **R-DAP-004** SHOULD: Apply TypeScript type assertions or runtime schema validation after parsing to enforce expected structure before access patterns execute.

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
- Parsing operations are wrapped in try-catch blocks with contextual error messages

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if configuration loading bypasses parsing validation. CI pipeline MUST fail if tests demonstrate runtime errors from unvalidated configuration access. Architecture review is REQUIRED for any new configuration loading patterns that deviate from parse-then-access flow.
</enforcement>