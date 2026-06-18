# Standardize Array.find() for Safe Data Access in Test Assertions: Test Utilities Provide

These rules are ALWAYS ACTIVE for all test files accessing parsed configuration data structures, including TOML-parsed MCP server lists and hierarchical ruler configurations.

### Rules

- **R-42-006** MAY: Test utilities MAY provide wrapper functions that combine Array.find() with assertion logic for common configuration access patterns.

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
- Reference tests (MistralVibeAgent.test.ts and apply-engine.test.ts) pass with no unsafe property access

<enforcement>
Clause Code MUST NOT skip or defer verification. All test files accessing parsed configuration data MUST be reviewed for compliance with Array.find() pattern usage and explicit undefined checks.
</enforcement>