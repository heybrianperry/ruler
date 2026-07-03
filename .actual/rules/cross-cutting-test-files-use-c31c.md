# Standardize Test Framework Structure with describe/it Blocks and JSON Validation: Test Files Use

These rules are ALWAYS ACTIVE for all test files in the codebase that process external configuration files and validate JSON data structures.

### Rules

- **R-TEST-001** MUST: All test files MUST use describe blocks to organize test suites with descriptive names that identify the feature or component under test.
- **R-TEST-002** MUST: All JSON.parse operations in test files MUST be wrapped with try-catch blocks or use helper functions that provide descriptive error messages for debugging.
- **R-TEST-003** MUST: Test files MUST use the standardized test harness utilities (setupTestProject, teardownTestProject, runRuler) from tests/harness.ts for consistent environment management.
- **R-TEST-004** SHOULD: Describe blocks SHOULD be structured hierarchically: top-level for the component/feature, nested for specific scenarios or behaviors.
- **R-TEST-005** SHOULD: Test names SHOULD read as specifications (e.g., 'should not create backup files when MCP is handled correctly') rather than generic descriptions.
- **R-TEST-006** SHOULD: File operations in tests SHOULD use async/await with fs/promises to avoid callback complexity.

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
```

**Accept when:**
- All test files in tests/ directory use describe/it block structure
- All JSON.parse operations in test files are wrapped with error handling or validation
- Test suite passes with 100% success rate in CI environment
- Code coverage for test harness utilities is above 80%
- No unhandled JSON parsing exceptions occur during test execution

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All test files must comply with R-TEST-001 through R-TEST-006 before code review approval.
</enforcement>