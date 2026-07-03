# Standardize on describe/it Test Structure with Jest-Compatible Framework: Nested Describe Blocks

These rules are ALWAYS ACTIVE for all test files in the codebase, including unit tests in `tests/unit/**/*.test.ts`, integration tests in `tests/integration/**/*.test.ts`, end-to-end tests in `tests/e2e/**/*.test.ts`, and all feature-specific test files in `tests/**/*.test.ts`.

### Rules

- **R-DESCRIBE-001** SHOULD: Use nested describe blocks to create hierarchical test organization when testing multiple aspects of a component or feature.
- **R-DESCRIBE-002** SHOULD: Name describe blocks with clear, descriptive strings that indicate the feature, component, or behavior being tested (e.g., 'MCP Backup Prevention for All Agents', 'Gemini MCP key usage').
- **R-DESCRIBE-003** SHOULD: Write it block descriptions as complete sentences starting with 'should' to clearly state expected behavior (e.g., 'should handle idempotent re-runs without creating backup files').
- **R-DESCRIBE-004** SHOULD: Place shared setup logic in beforeAll or beforeEach hooks, using beforeAll for expensive operations that can be shared across tests and beforeEach for test-specific state that must be isolated.
- **R-DESCRIBE-005** MUST: Always pair setup hooks with corresponding cleanup hooks (beforeAll with afterAll, beforeEach with afterEach) to ensure resources are properly released.
- **R-DESCRIBE-006** SHOULD: For integration tests with multiple verification steps, use nested describe blocks to organize related assertions while keeping individual it blocks focused on single concerns.
- **R-DESCRIBE-007** MAY: Specify custom timeouts for long-running integration tests as the third parameter to it blocks (e.g., `it('test name', async () => { ... }, 60000)`).
- **R-DESCRIBE-008** SHOULD: Limit nesting depth to a maximum of 3 levels; use test file splitting or helper functions to reduce complexity when deeper nesting is needed.

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
- Maximum nesting depth does not exceed 3 levels in any test file.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All new and modified test files MUST comply with R-DESCRIBE-001 through R-DESCRIBE-008 before approval.
</enforcement>