# Adopt Array.find() for Safe Configuration Lookup in Test Suites: Test Suites Use

These rules are ALWAYS ACTIVE for all test files validating parsed configuration arrays from TOML and JSON sources, including MistralVibeAgent.test.ts, apply-engine.test.ts, and similar test suites that query configuration objects by name, path, or identifier properties.

### Rules

- **R-CONFIG-LOOKUP-001** MUST: Test suites MUST use Array.find() with explicit predicate functions when querying parsed configuration arrays by name, path, or other identifying property.
- **R-CONFIG-LOOKUP-002** MUST: Test assertions MUST explicitly handle undefined return values from Array.find() before accessing properties (e.g., expect(server).toBeDefined() before expect(server.property).toBe(...)).
- **R-CONFIG-LOOKUP-003** MUST: Test files MUST NOT use direct array indexing (e.g., parsed.mcp_servers[0]) on parsed configuration structures; use Array.find() instead.
- **R-CONFIG-LOOKUP-004** SHOULD: Extract Array.find() results into named constants to improve test readability and enable reuse across multiple assertions within the same test case.
- **R-CONFIG-LOOKUP-005** SHOULD: Use path-based predicates for hierarchical configurations (e.g., configs.find((c) => c.rulerDir === expectedPath)).

### Verify

```bash
# Count Array.find() usage on parsed configuration arrays
grep -r 'parsed.*\.find(' tests/unit/ | grep -E '(mcp_servers|configs)' | wc -l

# Verify no direct indexing on parsed configuration structures
grep -r 'parsed.*\[0\]' tests/unit/ | grep -E '(mcp_servers|configs)' && echo 'Direct indexing found' || echo 'No direct indexing'

# Run configuration-related test suites with verbose output
npm test -- --testPathPattern='(MistralVibeAgent|apply-engine)\.test\.ts' --verbose
```

**Accept when:**
- All test files validating parsed configuration arrays use Array.find() with explicit predicates rather than direct indexing.
- Test assertions explicitly handle undefined return values from Array.find() before accessing properties.
- Grep verification shows no instances of direct array indexing (e.g., [0]) on parsed configuration structures in test files.
- Configuration lookup patterns are consistent across MistralVibeAgent.test.ts and apply-engine.test.ts test suites.

<enforcement>
Claude Code MUST NOT skip or defer verification. All test files in scope MUST be checked for Array.find() compliance before accepting changes. Direct indexing violations MUST block merge.
</enforcement>