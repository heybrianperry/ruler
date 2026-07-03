# Standardize Array.find() for Diagnostic and Entity Lookup in Test Suites: Diagnostic Configuration Result

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory that validate diagnostic messages, agent identifiers, and configuration results through collection searches.

### Rules

- **R-DIAG-001** SHOULD: Diagnostic and configuration result lookups SHOULD use Array.find() with code or identifier matching patterns.
- **R-DIAG-002** MUST: Always follow Array.find() calls with explicit undefined checks (e.g., `expect(result).toBeDefined()`) before accessing nested properties.
- **R-DIAG-003** SHOULD: For diagnostic validation, use the pattern: `config.diagnostics.find((d: any) => d.code === 'DIAGNOSTIC_CODE')`.
- **R-DIAG-004** SHOULD: For agent registry lookups, use the pattern: `allAgents.find((agent) => agent.getIdentifier() === 'identifier')`.
- **R-DIAG-005** SHOULD: Prefer Array.find() over Array.filter()[0] for single-entity lookups in test assertions.

### Verify

```bash
# Count Array.find() usage in diagnostic and agent lookups
grep -r '\.find(' tests/ | grep -E '(diagnostics\.find|allAgents\.find)' | wc -l

# Count Array.filter()[0] pattern usage (should be minimal)
grep -r '\.filter(.*\[0\]' tests/ | wc -l

# Run test suite for integration and unified-config tests
npm test -- --testPathPattern='(skills-mcp|unified-config)' --passWithNoTests
```

**Accept when:**
- Array.find() usage count in test files exceeds Array.filter()[0] pattern count by at least 3:1 ratio
- All diagnostic and agent lookup tests use Array.find() with explicit property comparisons
- Test suite passes with no undefined access errors from find() results
- All find() calls are followed by explicit undefined checks in assertions

<enforcement>
Claude Code MUST NOT skip or defer verification. All test files using diagnostic or agent lookups MUST conform to the Array.find() pattern with explicit undefined checks before proceeding.
</enforcement>