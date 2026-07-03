# Standardize Array.find() for Diagnostic and Entity Lookup in Test Suites: Tests Use Array

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory that perform entity lookups within collections returned by configuration loaders and agent registries.

### Rules

- **R-TEST-ARRAY-001** MUST: Use `Array.find()` with explicit property comparisons for single-entity lookup in test assertions, e.g., `collection.find(item => item.property === 'value')`.
- **R-TEST-ARRAY-002** MUST: Always follow `Array.find()` calls with explicit undefined checks before accessing nested properties, e.g., `expect(result).toBeDefined()`.
- **R-TEST-ARRAY-003** SHOULD: Use `Array.find()` for diagnostic validation with pattern: `config.diagnostics.find((d: any) => d.code === 'DIAGNOSTIC_CODE')`.
- **R-TEST-ARRAY-004** SHOULD: Use `Array.find()` for agent registry lookups with pattern: `allAgents.find((agent) => agent.getIdentifier() === 'identifier')`.
- **R-TEST-ARRAY-005** MAY: Use `Array.some()` when only existence verification is needed without entity access.

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
- `Array.find()` usage count in test files exceeds `Array.filter()[0]` pattern count by at least 3:1 ratio
- All diagnostic and agent lookup tests use `Array.find()` with explicit property comparisons
- Test suite passes with no undefined access errors from `find()` results
- All `find()` calls are followed by explicit undefined checks in assertions

<enforcement>
Clause Code MUST NOT skip or defer verification. All test files using entity lookups MUST conform to R-TEST-ARRAY-001 and R-TEST-ARRAY-002. Code review and ESLint enforcement are mandatory before merge.
</enforcement>