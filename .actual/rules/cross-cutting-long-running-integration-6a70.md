# Standardize on describe/it Test Structure with Jest-Compatible Framework: Long Running Integration

These rules are ALWAYS ACTIVE for all test files in the codebase, including unit tests in `tests/unit/**/*.test.ts`, integration tests in `tests/integration/**/*.test.ts`, end-to-end tests in `tests/e2e/**/*.test.ts`, and all feature-specific test files in `tests/**/*.test.ts`.

### Rules

- **R-JEST-001** MUST: Organize all test suites using `describe` blocks to group tests by feature, component, or behavior with clear, descriptive names (e.g., 'MCP Backup Prevention for All Agents').
- **R-JEST-002** MUST: Define all individual test cases using `it` blocks with descriptive names written as complete sentences starting with 'should' to clearly state expected behavior (e.g., 'should handle idempotent re-runs without creating backup files').
- **R-JEST-003** MUST: Always pair setup hooks with corresponding cleanup hooks (`beforeAll` with `afterAll`, `beforeEach` with `afterEach`) to ensure resources are properly released.
- **R-JEST-004** SHOULD: Place shared setup logic in `beforeAll` for expensive operations that can be shared across tests, and use `beforeEach` for test-specific state that must be isolated.
- **R-JEST-005** SHOULD: For integration tests with multiple verification steps, use nested `describe` blocks to organize related assertions while keeping individual `it` blocks focused on single concerns.
- **R-JEST-006** MAY: Long-running integration tests MAY specify custom timeout values using the third parameter to `it` blocks (e.g., `it('test name', async () => { ... }, 60000)`).
- **R-JEST-007** SHOULD: Limit nesting depth to a maximum of 3 levels; use test file splitting or helper functions to reduce complexity when deeper nesting is needed.

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
- Custom timeout values are present in long-running integration tests where needed
- Maximum nesting depth does not exceed 3 levels

<enforcement>
Claude Code MUST NOT skip or defer verification of test structure compliance. All new test files and modifications to existing test files MUST conform to these rules before approval.
</enforcement>