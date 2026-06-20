# Standardize on describe/it Test Framework Pattern for Test Organization: Test Files Use

These rules are ALWAYS ACTIVE for all test files in the codebase, including unit tests, integration tests, and functional tests.

### Rules

- **R-TEST-001** MUST: Use describe blocks to organize test suites, with describe blocks naming the component or feature under test.
- **R-TEST-002** MUST: Use it blocks for individual test cases with descriptive behavior-focused names starting with 'should'.
- **R-TEST-003** SHOULD: Use beforeEach and afterEach hooks for test setup and teardown to ensure proper test isolation.
- **R-TEST-004** MAY: Use beforeAll and afterAll hooks for expensive setup operations that can be shared across all tests in a suite.
- **R-TEST-005** SHOULD: Limit describe nesting to a maximum of 3 levels to maintain clarity and avoid obscuring test intent.
- **R-TEST-006** SHOULD: Keep test files focused on a single component or feature area to maintain clear organization.

### Verify

```bash
# Count describe blocks in test files
grep -r "describe(" tests/ | wc -l

# Count it blocks in test files
grep -r "it(" tests/ | wc -l

# Count lifecycle hooks in test files
grep -r "beforeEach\|afterEach" tests/ | wc -l
```

**Accept when:**
- All test files in tests/ directory use describe blocks for test suite organization
- Individual test cases are defined using it blocks with descriptive behavior-focused names
- Test files requiring setup/teardown use beforeEach/afterEach hooks appropriately
- describe blocks are nested no more than 3 levels deep
- Test file descriptions clearly express expected behavior

<enforcement>
Claude Code MUST NOT skip or defer verification. All new and modified test files MUST comply with these rules before merge. Violations are flagged during code review and developers must refactor tests to use the describe/it pattern. Exceptions require explicit justification in code comments and approval by tech lead or senior engineer.
</enforcement>