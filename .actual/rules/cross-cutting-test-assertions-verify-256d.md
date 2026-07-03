# Standardize Array.find() for Diagnostic and Entity Lookup in Test Suites: Test Assertions Verify

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory that validate diagnostic messages, agent identifiers, and configuration warnings through collection searches.

### Rules

- **R-TEST-ASSERT-001** SHOULD: Test assertions SHOULD verify the found entity is defined before accessing nested properties.
- **R-TEST-ASSERT-002** SHOULD: Use Array.find() with explicit property comparisons for single-entity lookup in test assertions: `collection.find(item => item.property === 'value')`.
- **R-TEST-ASSERT-003** SHOULD: Always follow find() calls with `expect(result).toBeDefined()` or similar assertions before accessing nested properties.
- **R-TEST-ASSERT-004** SHOULD: For diagnostic validation, use pattern: `config.diagnostics.find((d: any) => d.code === 'DIAGNOSTIC_CODE')`.
- **R-TEST-ASSERT-005** SHOULD: For agent registry lookups, use pattern: `allAgents.find((agent) => agent.getIdentifier() === 'identifier')`.

### Verify

```bash
# Count Array.find() usage in diagnostic and agent lookups
grep -r '\.find(' tests/ | grep -E '(diagnostics\.find|allAgents\.find)' | wc -l

# Count Array.filter()[0] pattern usage (should be minimal)
grep -r '\.filter(.*\[0\]' tests/ | wc -l

# Run test suite for integration and unit tests
npm test -- --testPathPattern='(skills-mcp|unified-config)' --passWithNoTests
```

**Accept when:**
- Array.find() usage count in test files exceeds Array.filter()[0] pattern count by at least 3:1 ratio
- All diagnostic and agent lookup tests use Array.find() with explicit property comparisons
- Test suite passes with no undefined access errors from find() results

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during test file review and modification.
</enforcement>