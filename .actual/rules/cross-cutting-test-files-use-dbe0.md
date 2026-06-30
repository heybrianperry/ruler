# Standardize on describe/it Test Structure with Jest-Compatible Framework: Test Files Use

These rules are ALWAYS ACTIVE for all test files in the codebase, including unit tests in `tests/unit/**/*.test.ts`, integration tests in `tests/integration/**/*.test.ts`, end-to-end tests in `tests/e2e/**/*.test.ts`, and all feature-specific test files in `tests/**/*.test.ts`.

### Rules

- **R-TEST-001** MUST: All test files MUST use describe blocks to organize related test cases into logical suites.
- **R-TEST-002** MUST: All test cases MUST be defined using it blocks with descriptive names that clearly state expected behavior.
- **R-TEST-003** MUST: Test files using temporary resources (file system, network) MUST include appropriate cleanup in afterAll or afterEach hooks.
- **R-TEST-004** SHOULD: Describe blocks SHOULD be named with clear, descriptive strings (e.g., 'MCP Backup Prevention for All Agents', 'Gemini MCP key usage').
- **R-TEST-005** SHOULD: It block descriptions SHOULD be written as complete sentences starting with 'should' to clearly state expected behavior.
- **R-TEST-006** SHOULD: Shared setup logic SHOULD be placed in beforeAll or beforeEach hooks, using beforeAll for expensive operations that can be shared across tests and beforeEach for test-specific state that must be isolated.
- **R-TEST-007** SHOULD: Setup hooks SHOULD always be paired with corresponding cleanup hooks (beforeAll with afterAll, beforeEach with afterEach) to ensure resources are properly released.
- **R-TEST-008** SHOULD: Integration tests with multiple verification steps SHOULD use nested describe blocks to organize related assertions while keeping individual it blocks focused on single concerns.
- **R-TEST-009** MAY: Tests requiring custom timeouts MAY specify timeout as third parameter: `it('test name', async () => { ... }, 60000)`.

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
- Maximum nesting depth of describe blocks does not exceed 3 levels

<enforcement>
Claude Code MUST NOT skip or defer verification of test file structure compliance. All new and modified test files MUST be checked against these rules before approval.
</enforcement>