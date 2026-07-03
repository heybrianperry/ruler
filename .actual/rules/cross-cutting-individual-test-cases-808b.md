# Standardize on describe/it Test Framework Pattern for Test Organization: Individual Test Cases

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory, including unit, integration, and functional tests.

### Rules

- **R-TEST-001** MUST: Individual test cases MUST be defined using `it` blocks with descriptive test names that clearly state the expected behavior.
- **R-TEST-002** MUST: Test suites MUST use `describe` blocks to organize and group related test cases hierarchically.
- **R-TEST-003** SHOULD: Write `it` block descriptions as complete sentences starting with 'should' to clearly express expected behavior (e.g., 'should return error when input is invalid').
- **R-TEST-004** SHOULD: Use `beforeEach` and `afterEach` hooks for common setup and cleanup logic to ensure test isolation.
- **R-TEST-005** SHOULD: Limit `describe` block nesting to a maximum of 3 levels to maintain clarity and avoid obscuring test intent.
- **R-TEST-006** MAY: Document exceptions to the describe/it pattern in test file headers with explicit rationale and expected duration.

### Verify

```bash
# Count describe blocks in test files
grep -r "describe(" tests/ | wc -l

# Count it blocks in test files
grep -r "it(" tests/ | wc -l

# Count beforeEach/afterEach hooks
grep -r "beforeEach\|afterEach" tests/ | wc -l
```

**Accept when:**
- All test files in `tests/` directory use `describe` blocks for test suite organization
- Individual test cases are defined using `it` blocks with descriptive behavior-focused names
- Test files requiring setup/teardown use `beforeEach`/`afterEach` hooks appropriately
- No test files use flat test function naming conventions without hierarchical organization
- `describe` block nesting does not exceed 3 levels in any test file

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules during code review and test file creation. All new test files MUST conform to the describe/it pattern before merge.
</enforcement>