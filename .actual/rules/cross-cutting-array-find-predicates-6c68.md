# Standardize Array.find() for Diagnostic and Entity Lookup in Test Suites: Array Find Predicates

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory that validate diagnostic messages, agent identifiers, and configuration warnings through collection searches.

### Rules

- **R-ARRAY-FIND-001** MUST: Array.find() predicates MUST use explicit property comparisons (e.g., `agent.getIdentifier() === 'value'`, `d.code === 'CODE'`) rather than implicit truthy checks or complex expressions.
- **R-ARRAY-FIND-002** MUST: All Array.find() calls in test assertions MUST be followed by explicit undefined checks (e.g., `expect(result).toBeDefined()`) before accessing nested properties.
- **R-ARRAY-FIND-003** SHOULD: Prefer Array.find() over Array.filter()[0] for single-entity lookups in test collections to improve readability and intent clarity.
- **R-ARRAY-FIND-004** SHOULD: Use Array.find() for diagnostic validation with pattern: `config.diagnostics.find((d: any) => d.code === 'DIAGNOSTIC_CODE')`.
- **R-ARRAY-FIND-005** SHOULD: Use Array.find() for agent registry lookups with pattern: `allAgents.find((agent) => agent.getIdentifier() === 'identifier')`.

### Verify

```bash
# Count Array.find() usage in test files
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
Clause Code MUST NOT skip or defer verification. All test files using collection searches for entity lookup MUST conform to these rules during code review and CI pipeline execution.
</enforcement>