# Standardize Test Framework Structure with describe/it Blocks and JSON Validation: Tests Verify Both

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory, including unit tests, integration tests, end-to-end tests, and test harness utilities.

### Rules

- **R-TEST-001** SHOULD: Tests SHOULD verify both positive cases (expected behavior) and negative cases (error handling, missing files, malformed data).
- **R-TEST-002** SHOULD: Test files SHOULD use describe/it block structure for organizing test suites.
- **R-TEST-003** SHOULD: JSON.parse operations in test files SHOULD be wrapped with error handling or validation.
- **R-TEST-004** SHOULD: Test files SHOULD use the standardized test harness (setupTestProject, teardownTestProject, runRuler) from tests/harness.ts for consistent environment management.
- **R-TEST-005** SHOULD: Describe blocks SHOULD be structured hierarchically: top-level for the component/feature, nested for specific scenarios or behaviors.
- **R-TEST-006** SHOULD: Test names SHOULD read as specifications and describe expected behavior clearly.

### Verify

```bash
# Count describe blocks in test files
grep -r "describe(" tests/ | wc -l

# Count it blocks in test files
grep -r "it(" tests/ | wc -l

# Count JSON.parse operations in test files
grep -r "JSON.parse" tests/ --include="*.test.ts" | wc -l

# Run test suite with coverage
npm test -- --coverage
```

**Accept when:**
- All test files in tests/ directory use describe/it block structure
- All JSON.parse operations in test files are wrapped with error handling or validation
- Test suite passes with 100% success rate in CI environment
- Code coverage for test harness utilities is above 80%
- Tests include both positive and negative test cases for configuration file operations

<enforcement>
Claude Code MUST NOT skip or defer verification of test structure compliance. All test files MUST follow the describe/it pattern and include error handling for JSON parsing operations.
</enforcement>