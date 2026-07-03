# Standardize Array.find() for Safe Data Access in Test Assertions: Predicate Functions Passed

These rules are ALWAYS ACTIVE for test files accessing parsed configuration data structures, including TOML-parsed MCP server lists, hierarchical ruler configurations, and JSON.parse() output validation in test assertions.

### Rules

- **R-42-005** SHOULD: Predicate functions passed to Array.find() SHOULD match on stable identifier properties (name, id, path) rather than positional indices.

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
- Reference test suites (MistralVibeAgent.test.ts, apply-engine.test.ts) pass with no unsafe property access warnings

<enforcement>
Clause R-42-005 verification is mandatory. Code review MUST flag direct array indexing patterns on parsed configuration objects and request refactor to Array.find() with explicit predicates. CI pipeline MUST run grep checks to detect violations. Test authors MUST document exceptions with EXC-ID references when deviating from this pattern.
</enforcement>