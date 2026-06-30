# Adopt Array.find() for Safe Configuration Lookup in Test Suites: Test Suites Not

These rules are ALWAYS ACTIVE for all test files validating parsed configuration structures from TOML and JSON sources, including MistralVibeAgent and apply-engine test suites.

### Rules

- **R-CONFIG-LOOKUP-001** SHOULD NOT: Test suites SHOULD NOT use direct array indexing (e.g., `array[0]`) to access parsed configuration entries unless array order is explicitly guaranteed by specification and documented in test comments.
- **R-CONFIG-LOOKUP-002** MUST: When querying configuration arrays by name, path, or identifier, use `Array.find()` with explicit predicates rather than direct indexing.
- **R-CONFIG-LOOKUP-003** MUST: Always assert that `Array.find()` results are defined before accessing properties: `expect(result).toBeDefined()` before `expect(result.property).toBe(value)`.
- **R-CONFIG-LOOKUP-004** SHOULD: Extract `Array.find()` results into named constants to improve test readability and enable reuse across multiple assertions within the same test case.
- **R-CONFIG-LOOKUP-005** SHOULD: Use named predicates that clearly express intent (e.g., `find((s) => s.name === 'existing_server')` rather than anonymous or positional lookups).

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
- All test files validating parsed configuration arrays use `Array.find()` with explicit predicates rather than direct indexing
- Test assertions explicitly handle undefined return values from `Array.find()` before accessing properties
- Grep verification shows no instances of direct array indexing (e.g., `[0]`) on parsed configuration structures in test files
- Configuration lookup patterns are consistent across MistralVibeAgent.test.ts and apply-engine.test.ts

<enforcement>
Claude Code MUST NOT skip or defer verification. All test files accessing parsed configuration arrays must use Array.find() with explicit undefined checks. Direct indexing violations must be flagged during code review and remediated before merge.
</enforcement>