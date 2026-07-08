# Standardize on describe/it Test Structure with Jest-Compatible Framework: Test Files That

These rules are ALWAYS ACTIVE for all TypeScript test files in the `tests/` directory, including unit tests, integration tests, and functional tests that verify agent behavior, configuration merging, and MCP server integration.

### Rules

- **R-TEST-001** MUST: Test files that create temporary project structures MUST clean up resources in afterEach or afterAll hooks using teardownTestProject or equivalent.
- **R-TEST-002** MUST: All test files in tests/ directory MUST use describe blocks for test suite organization.
- **R-TEST-003** MUST: All individual test cases MUST be defined using it blocks with descriptive names.
- **R-TEST-004** MUST: Test files requiring setup/teardown MUST use appropriate lifecycle hooks (beforeEach, afterEach, beforeAll, afterAll).
- **R-TEST-005** SHOULD: Use setupTestProject and teardownTestProject harness utilities for integration tests requiring temporary project structures.
- **R-TEST-006** SHOULD: Import core libraries (fs/promises, path, os) at the top of test files for consistent file system operations.
- **R-TEST-007** SHOULD: Use JSON.parse for configuration validation in tests that verify MCP server definitions, agent settings, and TOML-to-JSON transformations.
- **R-TEST-008** SHOULD: Organize test files by type: unit tests in tests/unit/, integration tests in tests/integration/, with descriptive filenames matching the component under test.
- **R-TEST-009** SHOULD: Use runRuler or runRulerWithInheritedStdio helper functions for tests that invoke the ruler CLI.

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
- Temporary project structures are cleaned up in afterEach or afterAll hooks

<enforcement>
Claude Code MUST NOT skip or defer verification of test structure conformance. All new test files and modifications to existing test files MUST be validated against these rules before acceptance.
</enforcement>