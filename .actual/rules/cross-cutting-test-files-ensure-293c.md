# Standardize on describe/it Test Structure with Jest-Compatible Framework: Test Files Ensure

These rules are ALWAYS ACTIVE for all test files in the codebase, including unit tests in `tests/unit/**/*.test.ts`, integration tests in `tests/integration/**/*.test.ts`, end-to-end tests in `tests/e2e/**/*.test.ts`, and all feature-specific test files in `tests/**/*.test.ts`.

### Rules

- **R-TEST-001** MUST: Test files MUST ensure proper cleanup of resources (temporary directories, file handles, mock state) in afterAll or afterEach hooks to prevent test pollution.
- **R-TEST-002** MUST: All test files MUST use describe blocks to group tests by feature, component, or behavior with clear, descriptive names.
- **R-TEST-003** MUST: All test cases MUST be defined using it blocks with descriptive names written as complete sentences starting with 'should'.
- **R-TEST-004** MUST: Shared setup logic MUST be placed in beforeAll or beforeEach hooks, with beforeAll used for expensive operations shared across tests and beforeEach for test-specific state requiring isolation.
- **R-TEST-005** MUST: Setup hooks MUST be paired with corresponding cleanup hooks (beforeAll with afterAll, beforeEach with afterEach) to ensure resources are properly released.
- **R-TEST-006** SHOULD: Integration tests with multiple verification steps SHOULD use nested describe blocks to organize related assertions while keeping individual it blocks focused on single concerns.
- **R-TEST-007** SHOULD: Deeply nested describe blocks SHOULD not exceed 3 levels of nesting; use test file splitting or helper functions to reduce complexity when deeper nesting is needed.
- **R-TEST-008** MAY: Tests requiring custom timeouts MAY specify timeout as third parameter: `it('test name', async () => { ... }, 60000)`.

### Verify

```bash
# Count describe blocks in test files
grep -r "describe(" tests/ | wc -l

# Count it blocks in test files
grep -r "it(" tests/ | wc -l

# Count lifecycle hook usage
grep -r "beforeAll\|beforeEach\|afterAll\|afterEach" tests/ | wc -l

# Verify all test files use describe blocks
npm test -- --listTests | xargs grep -L "describe(" || echo 'All test files use describe blocks'
```

**Accept when:**
- All test files in `tests/` directory contain at least one describe block organizing test cases
- All test cases are defined using it blocks with descriptive names
- Test files using temporary resources (file system, network) include appropriate cleanup in afterAll or afterEach hooks
- Code review confirms new test files follow the prescribed structure before merge
- No test files are missing describe blocks (verified by linting)
- Lifecycle hooks are properly paired (beforeAll with afterAll, beforeEach with afterEach)

<enforcement>
Claude Code MUST NOT skip or defer verification of test file structure compliance. All test files MUST be audited for adherence to describe/it patterns and proper resource cleanup before code review approval.
</enforcement>