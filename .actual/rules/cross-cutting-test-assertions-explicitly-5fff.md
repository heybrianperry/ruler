# Adopt Array.find() for Safe Configuration Lookup in Test Suites: Test Assertions Explicitly

These rules are ALWAYS ACTIVE for all test files validating parsed configuration data structures (TOML, JSON) containing MCP servers and hierarchical ruler configurations.

### Rules

- **R-CONFIG-001** MUST: Test assertions MUST explicitly handle the undefined return value from Array.find() when validating presence or absence of configuration entries.
- **R-CONFIG-002** MUST: When writing tests for parseTOML() or JSON.parse() results, immediately extract expected entries using Array.find() with named predicates.
- **R-CONFIG-003** MUST: Always assert on the find() result before accessing properties (e.g., expect(server).toBeDefined() before expect(server.property).toBe(...)).
- **R-CONFIG-004** SHOULD: For hierarchical configurations, use path-based predicates in Array.find() calls (e.g., configs.find((c) => c.rulerDir === expectedPath)).
- **R-CONFIG-005** SHOULD: Extract find() results into named constants to improve test readability and enable reuse across multiple assertions within the same test case.

### Verify

```bash
# Count Array.find() usage on configuration arrays
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
- All configuration lookup patterns in test suites follow the named predicate convention (e.g., find((s: any) => s.name === 'target_name')).

<enforcement>
Claude Code MUST NOT skip or defer verification. All test files matching the scope MUST comply with R-CONFIG-001 through R-CONFIG-005 before merge. Code review MUST validate explicit undefined handling on Array.find() results. CI pipeline MUST execute grep verification commands to detect violations.
</enforcement>