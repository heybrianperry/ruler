# Standardize on describe/it Test Structure with Jest-Compatible Framework: Test Files Use

These rules are ALWAYS ACTIVE for all TypeScript test files in the `tests/` directory, including unit tests, integration tests, and functional tests.

### Rules

- **R-TEST-001** MUST: All test files MUST use describe blocks to organize test suites with descriptive names indicating the component or feature under test.
- **R-TEST-002** MUST: All individual test cases MUST be defined using it blocks with descriptive names.
- **R-TEST-003** MUST: Test files requiring setup/teardown MUST use appropriate lifecycle hooks (beforeEach, afterEach, beforeAll, afterAll).
- **R-TEST-004** SHOULD: Use setupTestProject and teardownTestProject harness utilities for integration tests requiring temporary project structures.
- **R-TEST-005** SHOULD: Import core libraries (fs/promises, path, os) at the top of test files for consistent file system operations.
- **R-TEST-006** SHOULD: Use JSON.parse for configuration validation in tests that verify MCP server definitions and agent settings.
- **R-TEST-007** SHOULD: Organize test files by type: unit tests in tests/unit/, integration tests in tests/integration/, with descriptive filenames matching the component under test.
- **R-TEST-008** SHOULD: Use runRuler or runRulerWithInheritedStdio helper functions for tests that invoke the ruler CLI.

### Verify

```bash
# Count describe blocks in test files
grep -r "describe(" tests/ | wc -l

# Count it blocks in test files
grep -r "it(" tests/ | wc -l

# Count lifecycle hook usage
grep -r "beforeEach\|afterEach\|beforeAll\|afterAll" tests/ | wc -l

# Count test files matching pattern
npm test -- --listTests | grep -E '\.test\.ts$' | wc -l
```

**Accept when:**
- All test files in tests/ directory use describe blocks for test suite organization
- All individual test cases are defined using it blocks with descriptive names
- Test files requiring setup/teardown use appropriate lifecycle hooks (beforeEach, afterEach, beforeAll, afterAll)
- Test suite executes successfully with no hanging tests or resource leaks

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review process checks for describe/it structure in new test files. CI pipeline executes full test suite and reports test structure violations. ESLint rules enforce test naming conventions and lifecycle hook usage.
</enforcement>