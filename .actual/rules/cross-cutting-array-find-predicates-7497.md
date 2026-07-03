# Adopt Array.find() for Safe Configuration Lookup in Test Suites: Array Find Predicates

These rules are ALWAYS ACTIVE for all test files validating parsed configuration data structures (TOML, JSON) containing MCP servers and hierarchical ruler configurations, specifically within describe/it blocks using test frameworks.

### Rules

- **R-ARRAY-FIND-001** MUST: Array.find() predicates MUST compare against stable identifiers such as name, rulerDir, or path rather than array indices when querying configuration arrays in test suites.
- **R-ARRAY-FIND-002** MUST: Test assertions MUST explicitly handle undefined return values from Array.find() before accessing properties (e.g., expect(server).toBeDefined() before expect(server.property).toBe(...)).
- **R-ARRAY-FIND-003** SHOULD: Extract Array.find() results into named constants to improve test readability and enable reuse across multiple assertions within the same test case.
- **R-ARRAY-FIND-004** SHOULD: Use path-based predicates for hierarchical configurations (e.g., configs.find((c) => c.rulerDir === expectedPath)) rather than order-dependent access.

### Verify

```bash
# Count Array.find() usage on parsed configuration arrays
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
- Configuration lookup patterns use stable identifiers (name, rulerDir, path) as predicates, not array position

<enforcement>
Clause MUST NOT skip or defer verification. Code review MUST enforce the Array.find() pattern for configuration lookup in test files. CI pipeline MUST fail if grep verification detects direct array indexing on parsed configuration structures. Violations require remediation or documented exception with sign-off from test infrastructure owner.
</enforcement>