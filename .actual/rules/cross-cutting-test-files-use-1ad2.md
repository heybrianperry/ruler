# Standardize Test Framework Structure with describe/it Blocks and JSON Validation: Test Files Use

These rules are ALWAYS ACTIVE for all test files in the codebase that process external configuration files and validate JSON data structures.

### Rules

- **R-TEST-001** MAY: Test files MAY use nested describe blocks to organize related test cases into logical groupings.

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

<enforcement>
Claude Code MUST NOT skip or defer verification of test structure compliance. All test files MUST follow describe/it block organization and JSON parsing MUST include error handling.
</enforcement>