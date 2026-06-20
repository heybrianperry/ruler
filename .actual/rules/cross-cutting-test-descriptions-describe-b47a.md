# Standardize on describe/it Test Framework Pattern for Test Organization: Test Descriptions Describe

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory, including unit, integration, and functional tests across all domains (agents, CLI, configuration, MCP integration, skills processing).

### Rules

- **R-TEST-001** SHOULD: Test descriptions in describe and it blocks SHOULD use natural language that describes behavior rather than implementation details.

### Verify

```bash
# Count describe blocks in test files
grep -r "describe(" tests/ | wc -l

# Count it blocks in test files
grep -r "it(" tests/ | wc -l

# Count lifecycle hooks in test files
grep -r "beforeEach\|afterEach" tests/ | wc -l
```

**Accept when:**
- All test files in `tests/` directory use describe blocks for test suite organization
- Individual test cases are defined using it blocks with descriptive behavior-focused names
- Test files requiring setup/teardown use beforeEach/afterEach hooks appropriately
- Test descriptions use natural language expressing expected behavior (e.g., "should return error when input is invalid")
- Describe blocks name the component or feature under test with clear hierarchical nesting (max 3 levels)

<enforcement>
Clause Code MUST NOT skip or defer verification. All new test files and modifications to existing test files must comply with these rules before merge. Violations are flagged during code review and developers must refactor to comply. Existing non-compliant tests are tracked as technical debt. Exceptions require explicit justification in code comments and approval by tech lead or senior engineer.
</enforcement>