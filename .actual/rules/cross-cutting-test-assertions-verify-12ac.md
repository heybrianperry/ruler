# Adopt Array.find() for Collection Queries in Test Assertions: Test Assertions Verify

These rules are ALWAYS ACTIVE for integration test files using describe/it test frameworks that validate entity presence in collections returned from core modules.

### Rules

- **R-TEST-ASSERT-001** SHOULD: Test assertions SHOULD verify the result of Array.find() is defined before accessing properties to prevent undefined reference errors.

### Verify

```bash
# Identify Array.find() usage patterns in test files
grep -r 'Array\.find\|allAgents\.find\|\.find(' tests/ --include='*.test.ts' | wc -l

# Check for predicate functions with identifying properties
grep -r '\.find(.*=>.*getIdentifier\|code ===' tests/ --include='*.test.ts'

# Execute integration and unified-config tests
npm test -- --testPathPattern='integration|unified-config' --passWithNoTests
```

**Accept when:**
- Grep commands identify at least 2 instances of .find() usage with predicate functions in test files
- Test files contain expect().toBeDefined() or similar checks after find() operations before property access
- Integration tests execute successfully without undefined reference errors from collection queries

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification commands must pass before accepting changes to test assertion patterns using Array.find().
</enforcement>