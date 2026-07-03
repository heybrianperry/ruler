# Standardize Array.find() for Safe Data Access in Test Assertions: Test Code Use

These rules are ALWAYS ACTIVE for test files accessing parsed configuration data structures, including TOML-parsed MCP server lists and hierarchical ruler configurations.

### Rules

- **R-42-005** MUST: Test code MUST use Array.find() with explicit predicate functions when accessing elements from parsed configuration arrays.

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
- Reference tests (MistralVibeAgent.test.ts, apply-engine.test.ts) pass with all assertions intact

<enforcement>
Clause R-42-005 MUST be verified before test code accessing parsed configurations is merged. Code review MUST flag violations and request refactor to Array.find() pattern. CI pipeline MUST warn on direct array indexing in test files. Exceptions require documented justification with EXC-ID reference.
</enforcement>