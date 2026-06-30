# Adopt Array.find() for Safe Configuration Lookup in Test Suites: Test Suites Extract

These rules are ALWAYS ACTIVE for all test files validating parsed configuration structures from TOML and JSON sources, including MistralVibeAgent and apply-engine test suites.

### Rules

- **R-ARRAY-FIND-001** MAY: Test suites MAY extract Array.find() results into named constants (e.g., existingServer, rootConfig) to improve readability and enable reuse across multiple assertions.
- **R-ARRAY-FIND-002** MUST: When writing tests for parseTOML() or JSON.parse() results, immediately extract expected entries using Array.find() with named predicates rather than direct array indexing.
- **R-ARRAY-FIND-003** MUST: Always assert on the find() result before accessing properties: expect(server).toBeDefined(); expect(server.property).toBe(expectedValue).
- **R-ARRAY-FIND-004** SHOULD: For hierarchical configurations (e.g., ruler directories), use path-based predicates: configs.find((c) => c.rulerDir === expectedPath).

### Verify

```bash
# Count Array.find() usage on configuration arrays
grep -r 'parsed.*\.find(' tests/unit/ | grep -E '(mcp_servers|configs)' | wc -l

# Verify no direct indexing on parsed configuration structures
grep -r 'parsed.*\[0\]' tests/unit/ | grep -E '(mcp_servers|configs)' && echo 'Direct indexing found' || echo 'No direct indexing'

# Run configuration validation tests with verbose output
npm test -- --testPathPattern='(MistralVibeAgent|apply-engine)\.test\.ts' --verbose
```

**Accept when:**
- All test files validating parsed configuration arrays use Array.find() with explicit predicates rather than direct indexing
- Test assertions explicitly handle undefined return values from Array.find() before accessing properties
- Grep verification shows no instances of direct array indexing (e.g., [0]) on parsed configuration structures in test files
- All configuration lookup patterns in test suites follow the named constant extraction pattern for readability

<enforcement>
Claude Code MUST NOT skip or defer verification. All test files accessing parsed configuration arrays MUST use Array.find() with explicit undefined checks. Direct array indexing on parsed configuration structures is prohibited. Code review and CI pipeline verification are mandatory before merge.
</enforcement>