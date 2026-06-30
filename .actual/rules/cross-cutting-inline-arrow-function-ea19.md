# Adopt Array.find() for Diagnostic and Entity Lookup in Test Assertions: Inline Arrow Function

These rules are ALWAYS ACTIVE for test files that perform diagnostic message and entity object lookups within collection assertions.

### Rules

- **R-FIND-001** SHOULD: Inline arrow function predicates SHOULD be used for simple single-condition lookups to maintain test readability.

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

<enforcement>
Claude Code MUST NOT skip or defer verification. All test files using diagnostic or entity lookups MUST employ Array.find() with inline arrow function predicates and MUST include truthiness checks before property access.
</enforcement>