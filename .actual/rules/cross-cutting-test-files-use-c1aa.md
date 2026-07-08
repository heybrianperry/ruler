# Standardize on describe/it Test Structure with Jest-Compatible Framework: Test Files Use

These rules are ALWAYS ACTIVE for all TypeScript test files in the `tests/` directory, including unit tests, integration tests, and functional tests that validate agent behavior, configuration merging, and MCP server integration.

### Rules

- **R-TEST-001** SHOULD: Test files SHOULD use expect assertions for validation with descriptive matcher methods (toBe, toEqual, toBeDefined, toContain).
- **R-TEST-002** MUST: All test files MUST use describe blocks for test suite organization.
- **R-TEST-003** MUST: All individual test cases MUST be defined using it blocks with descriptive names.
- **R-TEST-004** SHOULD: Test files requiring setup/teardown SHOULD use appropriate lifecycle hooks (beforeEach, afterEach, beforeAll, afterAll).
- **R-TEST-005** SHOULD: Test files SHOULD import core libraries (fs/promises, path, os, child_process) at the top for consistent file system operations.
- **R-TEST-006** SHOULD: Configuration validation tests SHOULD use JSON.parse for MCP server definitions, agent settings, and TOML-to-JSON transformations.
- **R-TEST-007** SHOULD: Integration tests SHOULD use setupTestProject and teardownTestProject harness utilities for temporary project structures.
- **R-TEST-008** SHOULD: Tests invoking the ruler CLI SHOULD use runRuler or runRulerWithInheritedStdio helper functions for consistent command execution.

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
- expect assertions are used consistently with descriptive matcher methods throughout test files

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review process checks for describe/it structure in new test files. CI pipeline executes full test suite and reports test structure violations. ESLint rules enforce test naming conventions and lifecycle hook usage.
</enforcement>