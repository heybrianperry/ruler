# Adopt Array.find() for Collection Queries in Test Assertions: Collection Queries Performed

These rules are ALWAYS ACTIVE for integration test files using describe/it test frameworks that validate entity presence in collections returned from core modules.

### Rules

- **R-COLL-001** SHOULD: Collection queries SHOULD be performed immediately before assertions to maintain test readability and minimize variable scope.
- **R-COLL-002** MUST: Always verify the result of Array.find() with expect().toBeDefined() before accessing properties in assertions.
- **R-COLL-003** SHOULD: Use arrow functions for simple single-condition predicates; extract named functions for complex multi-condition logic.
- **R-COLL-004** SHOULD: Store found entities in descriptive variable names when multiple assertions are performed on the same entity.
- **R-COLL-005** SHOULD: Consider Array.filter() instead of find() when test needs to assert on multiple matching items or collection size.

### Verify

```bash
# Count Array.find() usage in test files
grep -r 'Array\.find\|allAgents\.find\|\.find(' tests/ --include='*.test.ts' | wc -l

# Identify find() usage with predicate functions
grep -r '\.find(.*=>.*getIdentifier\|code ===' tests/ --include='*.test.ts'

# Run integration and unified-config tests
npm test -- --testPathPattern='integration|unified-config' --passWithNoTests
```

**Accept when:**
- Grep commands identify at least 2 instances of .find() usage with predicate functions in test files
- Test files contain expect().toBeDefined() or similar checks after find() operations before property access
- Integration tests execute successfully without undefined reference errors from collection queries

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting changes to collection query patterns in test files.
</enforcement>