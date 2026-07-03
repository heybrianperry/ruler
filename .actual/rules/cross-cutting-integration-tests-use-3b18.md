# Standardize on describe/it Test Structure with Jest-Compatible Framework: Integration Tests Use

These rules are ALWAYS ACTIVE for all test files in the codebase, including unit tests in `tests/unit/**/*.test.ts`, integration tests in `tests/integration/**/*.test.ts`, end-to-end tests in `tests/e2e/**/*.test.ts`, and all feature-specific test files in `tests/**/*.test.ts`.

### Rules

- **R-JEST-001** SHOULD: Integration tests SHOULD use descriptive suite names that reflect the complete workflow being tested (e.g., 'ruler init → setup → apply → verify').
- **R-JEST-002** MUST: All test files MUST use describe blocks to group tests by feature, component, or behavior with clear, descriptive strings (e.g., 'MCP Backup Prevention for All Agents', 'Gemini MCP key usage').
- **R-JEST-003** MUST: All test cases MUST be defined using it blocks with descriptive names written as complete sentences starting with 'should' to clearly state expected behavior (e.g., 'should handle idempotent re-runs without creating backup files').
- **R-JEST-004** MUST: Shared setup logic MUST be placed in beforeAll or beforeEach hooks, using beforeAll for expensive operations that can be shared across tests and beforeEach for test-specific state that must be isolated.
- **R-JEST-005** MUST: Setup hooks MUST always be paired with corresponding cleanup hooks (beforeAll with afterAll, beforeEach with afterEach) to ensure resources are properly released.
- **R-JEST-006** SHOULD: Integration tests with multiple verification steps SHOULD use nested describe blocks to organize related assertions while keeping individual it blocks focused on single concerns.
- **R-JEST-007** MAY: Tests requiring custom timeouts (e.g., for long-running integration tests) MAY specify timeout as third parameter: `it('test name', async () => { ... }, 60000)`.
- **R-JEST-008** MUST: Test files using temporary resources (file system, network) MUST include appropriate cleanup in afterAll or afterEach hooks.
- **R-JEST-009** SHOULD: Deeply nested describe blocks SHOULD not exceed 3 levels of nesting; use test file splitting or helper functions to reduce complexity when deeper nesting is needed.

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
- All test files in `tests/` directory contain at least one describe block organizing test cases.
- All test cases are defined using it blocks with descriptive names starting with 'should'.
- Test files using temporary resources (file system, network) include appropriate cleanup in afterAll or afterEach hooks.
- Code review confirms new test files follow the prescribed structure before merge.
- No test files exist that lack describe blocks (except documented exceptions EXC-001 and EXC-002).

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All new and modified test files MUST comply with R-JEST-001 through R-JEST-009 before being considered complete. Violations MUST be flagged during code review and remediated before merge.
</enforcement>