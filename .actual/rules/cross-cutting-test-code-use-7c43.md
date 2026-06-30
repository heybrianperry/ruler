# Adopt Array.find() for Diagnostic and Entity Lookup in Test Assertions: Test Code Use

These rules are ALWAYS ACTIVE for all test code files that perform lookups of diagnostic messages and entity objects from collections within assertion contexts.

### Rules

- **R-TEST-FIND-001** MUST: Test code MUST use Array.find() when locating a single diagnostic or entity object from a collection for assertion purposes.

### Verify

```bash
# Count find() operations on diagnostic and entity collections
grep -r '\.find(' tests/ | grep -E '(diagnostics|agents|allAgents)' | wc -l

# Verify find() is used on diagnostic collections
grep -r 'config\.diagnostics\.find' tests/ --include='*.test.ts'

# Count truthiness assertions following find() operations
grep -r 'expect.*\.toBeTruthy()' tests/ --include='*.test.ts' | wc -l
```

**Accept when:**
- Test files contain .find() operations on diagnostic and entity collections with predicate functions matching on stable identifiers
- Assertions verify the found object is truthy before accessing nested properties
- Grep commands confirm the pattern appears consistently across test files in the tests/ directory
- No manual array iteration or filter()[0] patterns are used for single-element lookups in test assertions

<enforcement>
Claude Code MUST NOT skip or defer verification of this rule. All test code performing single-element lookups from collections must use Array.find() with appropriate truthiness checks before property access.
</enforcement>