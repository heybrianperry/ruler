# Adopt Array.find() for Diagnostic and Entity Lookup in Test Assertions: Test Assertions Verify

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory that perform diagnostic message and entity object lookups within test assertions.

### Rules

- **R-FIND-001** SHOULD: Test assertions SHOULD verify the found object is truthy before accessing nested properties to prevent undefined reference errors.
- **R-FIND-002** SHOULD: Use `const foundItem = collection.find(item => item.code === 'EXPECTED_CODE')` pattern for diagnostic lookups.
- **R-FIND-003** SHOULD: Always follow find() operations with `expect(foundItem).toBeTruthy()` or similar assertions before accessing properties.
- **R-FIND-004** SHOULD: Extract diagnostic codes and entity identifiers to constants at the top of test files for reusability and maintainability.
- **R-FIND-005** SHOULD: For agent lookups, use predicates like `agent.getIdentifier() === 'expected-id'` to match on stable identity methods.

### Verify

```bash
# Count find() operations on diagnostic and entity collections
grep -r '\.find(' tests/ | grep -E '(diagnostics|agents|allAgents)' | wc -l

# Find diagnostic collection find() patterns
grep -r 'config\.diagnostics\.find' tests/ --include='*.test.ts'

# Count truthiness assertions following find() operations
grep -r 'expect.*\.toBeTruthy()' tests/ --include='*.test.ts' | wc -l
```

**Accept when:**
- Test files contain `.find()` operations on diagnostic and entity collections with predicate functions matching on stable identifiers
- Assertions verify the found object is truthy before accessing nested properties
- Grep commands confirm the pattern appears consistently across test files in the `tests/` directory
- Diagnostic codes and entity identifiers are extracted to constants rather than hardcoded in predicates

<enforcement>
Claude Code MUST NOT skip or defer verification. All test files using diagnostic or entity lookups MUST follow the find() pattern with preceding truthiness checks.
</enforcement>