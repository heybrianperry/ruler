# Standardize on describe/it Test Structure with Jest-Compatible Framework: Individual Test Cases

These rules are ALWAYS ACTIVE for all test files in the codebase, including unit tests in `tests/unit/**/*.test.ts`, integration tests in `tests/integration/**/*.test.ts`, end-to-end tests in `tests/e2e/**/*.test.ts`, and all feature-specific test files in `tests/**/*.test.ts`.

### Rules

- **R-TEST-001** MUST: Individual test cases MUST be defined using `it` blocks with descriptive test names that clearly state the expected behavior.
- **R-TEST-002** MUST: Use `describe` blocks to group tests by feature, component, or behavior with clear, descriptive strings (e.g., 'MCP Backup Prevention for All Agents').
- **R-TEST-003** MUST: Write `it` block descriptions as complete sentences starting with 'should' to clearly state expected behavior (e.g., 'should handle idempotent re-runs without creating backup files').
- **R-TEST-004** MUST: Always pair setup hooks with corresponding cleanup hooks (`beforeAll` with `afterAll`, `beforeEach` with `afterEach`) to ensure resources are properly released.
- **R-TEST-005** SHOULD: Place shared setup logic in `beforeAll` or `beforeEach` hooks. Use `beforeAll` for expensive operations that can be shared across tests. Use `beforeEach` for test-specific state that must be isolated.
- **R-TEST-006** SHOULD: For integration tests with multiple verification steps, use nested `describe` blocks to organize related assertions while keeping individual `it` blocks focused on single concerns.
- **R-TEST-007** MAY: When tests require custom timeouts (e.g., for long-running integration tests), specify timeout as third parameter: `it('test name', async () => { ... }, 60000)`.
- **R-TEST-008** SHOULD: Establish guideline of maximum 3 levels of nesting in `describe` blocks. Use test file splitting or helper functions to reduce complexity when deeper nesting is needed.

### Verify

```bash
# Count describe blocks in test files
grep -r "describe(" tests/ | wc -l

# Count it blocks in test files
grep -r "it(" tests/ | wc -l

# Count lifecycle hooks
grep -r "beforeAll\|beforeEach\|afterAll\|afterEach" tests/ | wc -l

# Verify all test files use describe blocks
npm test -- --listTests | xargs grep -L "describe(" || echo 'All test files use describe blocks'
```

**Accept when:**
- All test files in `tests/` directory contain at least one `describe` block organizing test cases
- All test cases are defined using `it` blocks with descriptive names
- Test files using temporary resources (file system, network) include appropriate cleanup in `afterAll` or `afterEach` hooks
- Code review confirms new test files follow the prescribed structure before merge
- Maximum nesting depth does not exceed 3 levels in `describe` blocks

<enforcement>
Claude Code MUST NOT skip or defer verification of test structure compliance. All new test files and modifications to existing test files MUST adhere to these rules before approval.
</enforcement>