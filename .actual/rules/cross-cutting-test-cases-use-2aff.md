# Standardize Test Framework Structure with describe/it Blocks and JSON Validation: Test Cases Use

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory, including unit tests, integration tests, end-to-end tests, and test harness utilities.

### Rules

- **R-TEST-001** MUST: All test cases MUST use `it` blocks with clear, behavior-focused descriptions that explain the expected outcome.
- **R-TEST-002** MUST: All JSON.parse operations in test files MUST be wrapped with error handling (try-catch blocks) or validation helper functions that provide descriptive error messages.
- **R-TEST-003** MUST: Test files MUST use the standardized test harness utilities (setupTestProject, teardownTestProject, runRuler) from `tests/harness.ts` for consistent environment management.
- **R-TEST-004** SHOULD: Describe blocks SHOULD be structured hierarchically: top-level for the component/feature, nested for specific scenarios or behaviors.
- **R-TEST-005** SHOULD: Test names SHOULD read as specifications (e.g., 'should not create backup files when MCP is handled correctly') rather than generic descriptions.
- **R-TEST-006** MAY: Use `beforeAll`/`afterAll` hooks for expensive setup operations when performance optimization is needed.

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

# Verify all test files use describe/it structure
find tests/ -name "*.test.ts" -exec grep -L "describe(" {} \;

# Check for unhandled JSON.parse operations
grep -r "JSON.parse" tests/ --include="*.test.ts" | grep -v "try\|catch\|helper" || echo "All JSON.parse operations appear to have error handling"
```

**Accept when:**
- All test files in `tests/` directory use `describe`/`it` block structure
- All `JSON.parse` operations in test files are wrapped with error handling or validation
- Test suite passes with 100% success rate in CI environment
- Code coverage for test harness utilities is above 80%
- No test files are found without `describe` blocks
- All JSON parsing operations include try-catch or helper function wrapping

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All test files MUST comply with R-TEST-001 through R-TEST-006 before code review approval.
</enforcement>