# Standardize on describe/it Test Framework Pattern for Test Organization: Test Files Use

These rules are ALWAYS ACTIVE for all test files in the codebase, including unit tests, integration tests, and functional tests.

### Rules

- **R-TEST-001** MUST: All test files MUST use describe blocks to group related test cases into logical test suites.
- **R-TEST-002** MUST: Individual test cases MUST be defined using it blocks with descriptive behavior-focused names.
- **R-TEST-003** SHOULD: Test files requiring setup/teardown SHOULD use beforeEach/afterEach hooks appropriately for test isolation.
- **R-TEST-004** SHOULD: Use describe blocks to name the component or feature under test, with nested describe blocks for specific aspects or scenarios.
- **R-TEST-005** SHOULD: Write it block descriptions as complete sentences starting with 'should' to clearly express expected behavior.
- **R-TEST-006** MAY: Limit describe nesting to 3 levels maximum to maintain clarity and avoid overly complex test structures.

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
- describe blocks are used to name components or features under test
- it block descriptions express expected behavior clearly

<enforcement>
Claude Code MUST NOT skip or defer verification of test file organization against these rules.
</enforcement>