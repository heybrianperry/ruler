# Standardize Test Framework Structure with describe/it Blocks and JSON Validation: Tests That Verify

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory, including unit tests, integration tests, end-to-end tests, and test harness utilities.

### Rules

- **R-TEST-001** SHOULD: Tests that verify file operations SHOULD use the test harness utilities (setupTestProject, teardownTestProject, runRuler) for consistent environment management.
- **R-TEST-002** SHOULD: All test files SHOULD use describe/it block structure for organizing test suites.
- **R-TEST-003** SHOULD: JSON.parse operations in test files SHOULD be wrapped with try-catch blocks or helper functions that provide descriptive error messages.
- **R-TEST-004** SHOULD: describe blocks SHOULD be structured hierarchically: top-level for the component/feature, nested for specific scenarios or behaviors.
- **R-TEST-005** SHOULD: Test names SHOULD read as specifications (e.g., 'should not create backup files when MCP is handled correctly') rather than generic descriptions.
- **R-TEST-006** SHOULD: File operations in tests SHOULD use async/await with fs/promises and path modules for consistency.

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

# Verify test harness usage
grep -r "setupTestProject\|teardownTestProject\|runRuler" tests/ --include="*.test.ts" | wc -l
```

**Accept when:**
- All test files in `tests/` directory use describe/it block structure
- All JSON.parse operations in test files are wrapped with error handling or validation
- Test suite passes with 100% success rate in CI environment
- Code coverage for test harness utilities is above 80%
- File operations in tests use async/await patterns consistently
- Test names follow specification-style naming conventions

<enforcement>
Claude Code MUST NOT skip or defer verification of test structure compliance. All test files MUST conform to describe/it block organization and JSON validation patterns before code review approval.
</enforcement>