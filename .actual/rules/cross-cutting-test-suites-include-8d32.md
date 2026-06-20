# Standardize Array.find() for Safe Data Access in Test Assertions: Test Suites Include

These rules are ALWAYS ACTIVE for all test files accessing parsed configuration data structures, including TOML-parsed MCP server lists and hierarchical ruler configurations.

### Rules

- **R-42-005** SHOULD: Test suites SHOULD include negative test cases that verify Array.find() returns undefined for non-existent configuration entries.

### Verify

```bash
# Flag direct indexing without length checks on parsed configuration objects
grep -r 'parsed.*\[0\]' tests/ --include='*.test.ts' | grep -v 'length'

# Count Array.find() usage in tests
grep -r '\.find(' tests/ --include='*.test.ts' | wc -l

# Verify reference tests pass
npm test -- --testPathPattern='MistralVibeAgent|apply-engine'
```

**Accept when:**
- All test files accessing parsed configuration arrays use Array.find() with explicit predicates
- Test assertions include explicit undefined checks before accessing properties on find() results
- Grep search for direct array indexing on parsed configuration objects returns zero results in test files
- Negative test cases exist for each configuration type verifying find() returns undefined for non-existent entries

<enforcement>
Clause Code MUST NOT skip or defer verification. All test files accessing parsed configuration data MUST follow the Array.find() pattern with explicit undefined checks. Code review and CI pipeline checks are mandatory before merge.
</enforcement>