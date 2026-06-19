# Standardize Test Framework Structure with describe/it Blocks and JSON Validation: Test Files That

These rules are ALWAYS ACTIVE for all test files in the codebase that process external configuration files and validate JSON data structures.

### Rules

- **R-TEST-001** MUST: Test files that read configuration files MUST validate the structure and required properties of parsed JSON objects before making assertions.
- **R-TEST-002** MUST: All JSON.parse operations in test files MUST be wrapped with try-catch blocks or use helper functions that provide descriptive error messages for debugging.
- **R-TEST-003** MUST: Test files MUST use describe/it block structure for organizing test suites to improve readability and maintainability.
- **R-TEST-004** MUST: Test files that modify filesystem state MUST use the standardized test harness (setupTestProject, teardownTestProject, runRuler) from tests/harness.ts for consistent environment management and isolation.
- **R-TEST-005** SHOULD: Describe blocks SHOULD be structured hierarchically: top-level for the component/feature, nested for specific scenarios or behaviors.
- **R-TEST-006** SHOULD: Test names SHOULD read as specifications (e.g., 'should not create backup files when MCP is handled correctly') rather than generic descriptions.

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

# Verify no unhandled JSON.parse operations
grep -r "JSON.parse" tests/ --include="*.test.ts" -B 2 -A 2 | grep -E "(try|catch|helper)"
```

**Accept when:**
- All test files in tests/ directory use describe/it block structure
- All JSON.parse operations in test files are wrapped with error handling or validation
- Test suite passes with 100% success rate in CI environment
- Code coverage for test harness utilities is above 80%
- All test files use setupTestProject/teardownTestProject for filesystem operations

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All test files must comply with R-TEST-001 through R-TEST-006 before code review approval.
</enforcement>