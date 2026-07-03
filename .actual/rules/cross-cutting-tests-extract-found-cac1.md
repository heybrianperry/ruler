# Adopt Array.find() for Diagnostic and Entity Lookup in Test Assertions: Tests Extract Found

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory that perform lookups on diagnostic and entity collections.

### Rules

- **R-FIND-001** MAY: Tests MAY extract the found object into a named variable when multiple assertions are performed on the same entity.

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
- Test files contain `.find()` operations on diagnostic and entity collections with predicate functions matching on stable identifiers (e.g., diagnostic codes like 'MCP_JSON_DEPRECATED', agent identifiers like 'jules')
- Assertions verify the found object is truthy before accessing nested properties
- Grep commands confirm the pattern appears consistently across test files in the tests/ directory
- Diagnostic codes and entity identifiers are extracted to constants at the top of test files for reusability
- Predicates use stable identity methods (e.g., `agent.getIdentifier() === 'expected-id'`) rather than positional or volatile attributes

<enforcement>
Claude Code MUST NOT skip or defer verification. All test files using find() for diagnostic or entity lookup MUST include truthiness checks before property access. Code review MUST flag manual iteration or filter()[0] patterns and request refactoring to use find().
</enforcement>