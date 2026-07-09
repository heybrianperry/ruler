# Standardize on describe/it Test Structure with Jest-Compatible Framework: Test Files Use

These rules are ALWAYS ACTIVE for all TypeScript test files in the `tests/` directory, including unit tests, integration tests, and functional tests.

### Rules

- **R-TEST-001** MUST: Test files MUST use beforeEach/afterEach hooks for setup and teardown when test isolation requires state management.
- **R-TEST-002** MUST: All test files MUST use describe blocks for test suite organization.
- **R-TEST-003** MUST: All individual test cases MUST be defined using it blocks with descriptive names.
- **R-TEST-004** SHOULD: Test files SHOULD use setupTestProject and teardownTestProject harness utilities for integration tests requiring temporary project structures.
- **R-TEST-005** SHOULD: Test files SHOULD import core libraries (fs/promises, path, os, child_process) at the top for consistent file system operations.
- **R-TEST-006** SHOULD: Test files SHOULD use JSON.parse for configuration validation when verifying MCP server definitions, agent settings, and TOML-to-JSON transformations.
- **R-TEST-007** SHOULD: Test files SHOULD use runRuler or runRulerWithInheritedStdio helper functions for tests that invoke the ruler CLI.

### Verify

```bash
# Count describe blocks in test files
grep -r "describe(" tests/ | wc -l

# Count it blocks in test files
grep -r "it(" tests/ | wc -l

# Count lifecycle hook usage
grep -r "beforeEach\|afterEach\|beforeAll\|afterAll" tests/ | wc -l

# Count test files
npm test -- --listTests | grep -E '\.test\.ts$' | wc -l
```

**Accept when:**
- All test files in tests/ directory use describe blocks for test suite organization
- All individual test cases are defined using it blocks with descriptive names
- Test files requiring setup/teardown use appropriate lifecycle hooks (beforeEach, afterEach, beforeAll, afterAll)
- Test suite executes successfully with no hanging tests or resource leaks

<enforcement>
Claude Code MUST NOT skip or defer verification of test file structure compliance.
</enforcement>