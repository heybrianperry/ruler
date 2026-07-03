# Adopt Array.find() for Safe Configuration Lookup in Test Suites: Configuration Parsing Operations

These rules are ALWAYS ACTIVE for test files validating parsed configuration structures from TOML and JSON sources, particularly in test suites for MistralVibeAgent and apply-engine that query arrays of server or configuration objects by name or path.

### Rules

- **R-CONFIG-001** SHOULD: Configuration parsing operations (parseTOML, JSON.parse) SHOULD be followed immediately by Array.find() lookups in test validation blocks to minimize scope of unsafe references.
- **R-CONFIG-002** MUST: Test assertions querying configuration arrays by name, path, or identifier properties MUST use Array.find() with explicit predicates rather than direct array indexing.
- **R-CONFIG-003** MUST: Array.find() results MUST be explicitly checked for undefined before accessing properties in test assertions.
- **R-CONFIG-004** SHOULD: Configuration lookup results SHOULD be extracted into named constants to improve test readability and enable reuse across multiple assertions.

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
- All test files validating parsed configuration arrays use Array.find() with explicit predicates rather than direct indexing
- Test assertions explicitly handle undefined return values from Array.find() before accessing properties
- Grep verification shows no instances of direct array indexing (e.g., [0]) on parsed configuration structures in test files
- Configuration lookup patterns are consistent across MistralVibeAgent.test.ts and apply-engine.test.ts test suites

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration array access in test validation blocks MUST use Array.find() with explicit undefined checks. Direct indexing violations MUST be flagged during code review and CI pipeline checks MUST fail if direct indexing is detected on parsed configuration structures.
</enforcement>