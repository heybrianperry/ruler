# Standardize Array.find() for Safe Data Access in Test Assertions: Test Assertions Explicitly

These rules are ALWAYS ACTIVE for test files accessing parsed configuration data structures, including TOML-parsed MCP server lists and hierarchical ruler configurations.

### Rules

- **R-42-005** MUST: Test assertions MUST explicitly check for undefined results from Array.find() before accessing properties on the returned object.

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
- Reference tests (MistralVibeAgent.test.ts and apply-engine.test.ts) pass with all assertions properly guarded

<enforcement>
Clause R-42-005 MUST be verified in all test files accessing parsed TOML configuration data, hierarchical configuration structures, or JSON.parse() output. Claude Code MUST NOT skip or defer verification of explicit undefined checks before property access on Array.find() results.
</enforcement>