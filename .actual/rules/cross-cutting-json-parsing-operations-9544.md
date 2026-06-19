# Standardize Test Framework Structure with describe/it Blocks and JSON Validation: Json Parsing Operations

These rules are ALWAYS ACTIVE for all test files in the codebase that process JSON configuration data and validate MCP server configurations.

### Rules

- **R-JSON-001** MUST: All JSON parsing operations MUST use JSON.parse with explicit error handling to prevent unhandled exceptions from malformed input.

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
Claude Code MUST NOT skip or defer verification of JSON parsing error handling in test files. All JSON.parse operations must be wrapped in try-catch blocks or use helper functions with descriptive error messages.
</enforcement>