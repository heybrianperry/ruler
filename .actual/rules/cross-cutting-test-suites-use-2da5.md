# Standardize Test Framework Structure with describe/it Blocks and JSON Validation: Test Suites Use

These rules are ALWAYS ACTIVE for all test files in the codebase, including unit tests, integration tests, end-to-end tests, and test harness utilities.

### Rules

- **R-TEST-001** SHOULD: Test suites SHOULD use beforeEach/beforeAll and afterEach/afterAll hooks to set up and tear down test environments, ensuring test isolation.
- **R-TEST-002** SHOULD: Test suites SHOULD use describe/it block structure for organizing test cases, with hierarchical nesting for components/features and specific scenarios.
- **R-TEST-003** SHOULD: JSON.parse operations in test files SHOULD be wrapped with try-catch blocks or helper functions that provide descriptive error messages for debugging.
- **R-TEST-004** SHOULD: Test names SHOULD read as specifications describing expected behavior (e.g., 'should not create backup files when MCP is handled correctly').
- **R-TEST-005** SHOULD: Test files SHOULD use the standardized test harness utilities (setupTestProject, teardownTestProject, runRuler) from tests/harness.ts for consistent environment management.
- **R-TEST-006** SHOULD: File operations in tests SHOULD use async/await with fs/promises and path modules consistently to avoid callback complexity.

### Verify

```bash
# Count describe blocks in test files
grep -r "describe(" tests/ | wc -l

# Count it blocks in test files
grep -r "it(" tests/ | wc -l

# Count JSON.parse operations in test files
grep -r "JSON.parse" tests/ --include="*.test.ts" | wc -l

# Run full test suite with coverage
npm test -- --coverage

# Verify test harness usage
grep -r "setupTestProject\|teardownTestProject\|runRuler" tests/ --include="*.test.ts" | wc -l
```

**Accept when:**
- All test files in tests/ directory use describe/it block structure
- All JSON.parse operations in test files are wrapped with error handling or validation
- Test suite passes with 100% success rate in CI environment
- Code coverage for test harness utilities is above 80%
- Test files consistently use setupTestProject, teardownTestProject, and runRuler from the harness

<enforcement>
Claude Code MUST NOT skip or defer verification of test structure compliance. All test files MUST follow the describe/it pattern with proper setup/teardown hooks and JSON validation.
</enforcement>