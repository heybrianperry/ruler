# Standardize on describe/it Test Structure with Jest-Compatible Framework: Test Files Nest

These rules are ALWAYS ACTIVE for all TypeScript test files in the `tests/` directory, including unit tests, integration tests, and functional tests.

### Rules

- **R-TEST-001** SHOULD: Test files SHOULD nest describe blocks to create hierarchical test organization when testing multiple aspects of a component.
- **R-TEST-002** MUST: All test files in `tests/` directory MUST use describe blocks for test suite organization.
- **R-TEST-003** MUST: All individual test cases MUST be defined using it blocks with descriptive names.
- **R-TEST-004** SHOULD: Test files requiring setup/teardown SHOULD use appropriate lifecycle hooks (beforeEach, afterEach, beforeAll, afterAll).
- **R-TEST-005** SHOULD: Test files SHOULD import core libraries (fs/promises, path, os, child_process) at the top for consistent file system operations.
- **R-TEST-006** SHOULD: Test files SHOULD use JSON.parse for configuration validation in tests that verify MCP server definitions, agent settings, and TOML-to-JSON transformations.
- **R-TEST-007** SHOULD: Integration tests SHOULD use setupTestProject and teardownTestProject harness utilities for temporary project structures.
- **R-TEST-008** SHOULD: Tests invoking the ruler CLI SHOULD use runRuler or runRulerWithInheritedStdio helper functions for consistent command execution and output capture.

### Verify

```bash
# Count describe blocks
grep -r "describe(" tests/ | wc -l

# Count it blocks
grep -r "it(" tests/ | wc -l

# Count lifecycle hooks
grep -r "beforeEach\|afterEach\|beforeAll\|afterAll" tests/ | wc -l

# Count test files
npm test -- --listTests | grep -E '\.test\.ts$' | wc -l
```

**Accept when:**
- All test files in `tests/` directory use describe blocks for test suite organization
- All individual test cases are defined using it blocks with descriptive names
- Test files requiring setup/teardown use appropriate lifecycle hooks (beforeEach, afterEach, beforeAll, afterAll)
- Test suite executes successfully with no hanging tests or resource leaks

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review process checks for describe/it structure in new test files. CI pipeline executes full test suite and reports test structure violations. ESLint rules enforce test naming conventions and lifecycle hook usage.
</enforcement>