# Standardize Array.find() for Diagnostic and Entity Lookup in Test Suites: Test Code Use

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory that validate diagnostic messages, agent identifiers, and configuration warnings through collection searches.

### Rules

- **R-TEST-001** MUST: Test code MUST use Array.find() when locating a single entity by property match within a collection.
- **R-TEST-002** MUST: Always follow find() calls with expect(result).toBeDefined() or similar assertions before accessing nested properties.
- **R-TEST-003** SHOULD: Use explicit property comparisons in find() predicates: `collection.find(item => item.property === 'value')`.
- **R-TEST-004** SHOULD: For diagnostic validation, use pattern: `config.diagnostics.find((d: any) => d.code === 'DIAGNOSTIC_CODE')`.
- **R-TEST-005** SHOULD: For agent registry lookups, use pattern: `allAgents.find((agent) => agent.getIdentifier() === 'identifier')`.

### Verify

```bash
# Count Array.find() usage in diagnostic and agent lookups
grep -r '\.find(' tests/ | grep -E '(diagnostics\.find|allAgents\.find)' | wc -l

# Count Array.filter()[0] pattern usage (should be minimal)
grep -r '\.filter(.*\[0\]' tests/ | wc -l

# Run test suite to verify no undefined access errors
npm test -- --testPathPattern='(skills-mcp|unified-config)' --passWithNoTests
```

**Accept when:**
- Array.find() usage count in test files exceeds Array.filter()[0] pattern count by at least 3:1 ratio
- All diagnostic and agent lookup tests use Array.find() with explicit property comparisons
- Test suite passes with no undefined access errors from find() results
- No filter()[0] patterns remain in test entity lookup code

<enforcement>
Clause Code MUST NOT skip or defer verification. All test files using collection searches for entity lookup MUST conform to R-TEST-001 through R-TEST-005. Violations detected during code review or CI execution are blocking and require immediate refactoring.
</enforcement>