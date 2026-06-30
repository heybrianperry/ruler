# Standardize on describe/it Test Structure with Jest-Compatible Framework: Test Assertions Use

These rules are ALWAYS ACTIVE for all test files in the codebase, including unit tests in `tests/unit/**/*.test.ts`, integration tests in `tests/integration/**/*.test.ts`, end-to-end tests in `tests/e2e/**/*.test.ts`, and all feature-specific test files in `tests/**/*.test.ts`.

### Rules

- **R-TEST-ASSERT-001** MUST: Test assertions MUST use the expect API with appropriate matchers (toBe, toEqual, toContain, toBeDefined, etc.) for clear failure messages.

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

# Check for expect() usage in test files
grep -r "expect(" tests/ | wc -l
```

**Accept when:**
- All test files in `tests/` directory contain at least one `describe` block organizing test cases
- All test cases are defined using `it` blocks with descriptive names
- Test assertions consistently use the `expect()` API with appropriate matchers
- Test files using temporary resources (file system, network) include appropriate cleanup in `afterAll` or `afterEach` hooks
- Code review confirms new test files follow the prescribed structure before merge

<enforcement>
Claude Code MUST NOT skip or defer verification of test assertion patterns. All test files MUST use the expect API with appropriate matchers as specified in R-TEST-ASSERT-001.
</enforcement>