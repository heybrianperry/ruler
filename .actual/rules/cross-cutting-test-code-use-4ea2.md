# Adopt Array.find() for Collection Queries in Test Assertions: Test Code Use

These rules are ALWAYS ACTIVE for integration test files using describe/it test frameworks that validate entity presence in collections returned from core modules.

### Rules

- **R-TEST-FIND-001** MUST: Test code MUST use Array.find() with predicate functions when querying collections to locate a single matching entity.
- **R-TEST-FIND-002** MUST: Always verify the result of Array.find() with expect().toBeDefined() before accessing properties in assertions.
- **R-TEST-FIND-003** SHOULD: Use arrow functions for simple single-condition predicates; extract named functions for complex multi-condition logic.
- **R-TEST-FIND-004** SHOULD: Store found entities in descriptive variable names when multiple assertions are performed on the same entity.
- **R-TEST-FIND-005** MAY: Consider Array.filter() instead of find() when test needs to assert on multiple matching items or collection size.

### Verify

```bash
# Count Array.find() usage in test files
grep -r 'Array\.find\|allAgents\.find\|\.find(' tests/ --include='*.test.ts' | wc -l

# Identify find() usage with predicate functions
grep -r '\.find(.*=>.*getIdentifier\|code ===' tests/ --include='*.test.ts'

# Execute integration and unified-config tests
npm test -- --testPathPattern='integration|unified-config' --passWithNoTests
```

**Accept when:**
- Grep commands identify at least 2 instances of .find() usage with predicate functions in test files
- Test files contain expect().toBeDefined() or similar checks after find() operations before property access
- Integration tests execute successfully without undefined reference errors from collection queries

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting changes to test code using Array.find().
</enforcement>