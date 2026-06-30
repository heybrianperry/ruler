# Standardize on describe/it Test Structure with Jest-Compatible Framework: Test Setup Teardown

These rules are ALWAYS ACTIVE for all test files in the codebase, including unit tests in `tests/unit/**/*.test.ts`, integration tests in `tests/integration/**/*.test.ts`, end-to-end tests in `tests/e2e/**/*.test.ts`, and all feature-specific test files in `tests/**/*.test.ts`.

### Rules

- **R-TEST-001** MUST: Test setup and teardown logic MUST use appropriate lifecycle hooks: `beforeAll` for suite-level setup, `beforeEach` for test-level setup, `afterAll` for suite-level cleanup, and `afterEach` for test-level cleanup.
- **R-TEST-002** MUST: All test files MUST use `describe` blocks to group tests by feature, component, or behavior with clear, descriptive names (e.g., 'MCP Backup Prevention for All Agents').
- **R-TEST-003** MUST: All test cases MUST be defined using `it` blocks with descriptive names written as complete sentences starting with 'should' to clearly state expected behavior.
- **R-TEST-004** MUST: Setup hooks MUST always be paired with corresponding cleanup hooks (`beforeAll` with `afterAll`, `beforeEach` with `afterEach`) to ensure resources are properly released.
- **R-TEST-005** SHOULD: For integration tests with multiple verification steps, consider using nested `describe` blocks to organize related assertions while keeping individual `it` blocks focused on single concerns, with a maximum nesting depth of 3 levels.
- **R-TEST-006** SHOULD: Use `beforeAll` for expensive operations that can be shared across tests (e.g., creating test projects) and `beforeEach` for test-specific state that must be isolated.
- **R-TEST-007** MAY: When tests require custom timeouts (e.g., for long-running integration tests), specify timeout as third parameter: `it('test name', async () => { ... }, 60000)`.

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
- All test files in `tests/` directory contain at least one `describe` block organizing test cases
- All test cases are defined using `it` blocks with descriptive names starting with 'should'
- Test files using temporary resources (file system, network) include appropriate cleanup in `afterAll` or `afterEach` hooks
- Code review confirms new test files follow the prescribed structure before merge
- Maximum nesting depth of `describe` blocks does not exceed 3 levels
- All lifecycle hooks are properly paired (beforeAll with afterAll, beforeEach with afterEach)

<enforcement>
Claude Code MUST NOT skip or defer verification of test structure compliance. All new test files and modifications to existing test files MUST be verified against these rules before approval.
</enforcement>