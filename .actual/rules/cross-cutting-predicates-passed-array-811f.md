# Adopt Array.find() for Diagnostic and Entity Lookup in Test Assertions: Predicates Passed Array

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory that perform diagnostic message and entity object lookups within test assertions.

### Rules

- **R-PRED-001** MUST: Predicates passed to Array.find() MUST match on stable identifiers such as diagnostic codes, entity identifiers, or unique property values.
- **R-PRED-002** MUST: Always follow find() operations with expect(foundItem).toBeTruthy() or similar assertions before accessing properties on the found object.
- **R-PRED-003** SHOULD: Extract diagnostic codes and entity identifiers to constants at the top of test files for reusability and maintainability.
- **R-PRED-004** SHOULD: Keep predicates simple and single-purpose; add debug logging or intermediate variables for complex lookup conditions.

### Verify

```bash
# Count find() operations on diagnostic and entity collections
grep -r '\.find(' tests/ | grep -E '(diagnostics|agents|allAgents)' | wc -l

# Verify config.diagnostics.find pattern usage
grep -r 'config\.diagnostics\.find' tests/ --include='*.test.ts'

# Count truthiness assertions following find operations
grep -r 'expect.*\.toBeTruthy()' tests/ --include='*.test.ts' | wc -l
```

**Accept when:**
- Test files contain .find() operations on diagnostic and entity collections with predicate functions matching on stable identifiers
- Assertions verify the found object is truthy before accessing nested properties
- Grep commands confirm the pattern appears consistently across test files in the tests/ directory
- Diagnostic codes and entity identifiers are extracted to constants or inline predicates use stable identity methods

<enforcement>
Claude Code MUST NOT skip or defer verification. All test files using Array.find() for diagnostic or entity lookup MUST comply with R-PRED-001 and R-PRED-002. Code review and linting must enforce these rules before merge.
</enforcement>