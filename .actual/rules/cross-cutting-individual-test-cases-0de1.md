# Standardize on describe/it Test Structure with Jest-Compatible Framework: Individual Test Cases

These rules are ALWAYS ACTIVE for all TypeScript test files in the tests/ directory, including unit tests, integration tests, and functional tests.

### Rules

- **R-TEST-001** MUST: Individual test cases MUST be defined using it blocks with clear, behavior-describing names.

### Verify

```bash
# Count describe blocks in test files
grep -r "describe(" tests/ | wc -l

# Count it blocks in test files
grep -r "it(" tests/ | wc -l

# Count lifecycle hooks usage
grep -r "beforeEach\|afterEach\|beforeAll\|afterAll" tests/ | wc -l

# Count test files matching pattern
npm test -- --listTests | grep -E '\.test\.ts$' | wc -l
```

**Accept when:**
- All test files in tests/ directory use describe blocks for test suite organization
- All individual test cases are defined using it blocks with descriptive names
- Test files requiring setup/teardown use appropriate lifecycle hooks (beforeEach, afterEach, beforeAll, afterAll)
- Test suite executes successfully with no hanging tests or resource leaks

<enforcement>
Clause Code MUST NOT skip or defer verification. All new test files and modifications to existing test files must conform to the describe/it structure with descriptive test case names before merge.
</enforcement>