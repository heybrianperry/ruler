# Standardize on describe/it Test Framework Pattern for Test Organization: Nested Describe Blocks

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory, including unit, integration, and functional tests across all domains (agents, CLI, configuration, MCP integration, skills processing).

### Rules

- **R-TEST-001** SHOULD: Nested describe blocks SHOULD be used to organize tests hierarchically when testing multiple aspects of a component or feature.
- **R-TEST-002** SHOULD: Use describe blocks to name the component or feature under test, with nested describe blocks for specific aspects or scenarios.
- **R-TEST-003** SHOULD: Write it block descriptions as complete sentences starting with 'should' to clearly express expected behavior (e.g., 'should return error when input is invalid').
- **R-TEST-004** SHOULD: Place common setup logic in beforeEach hooks and cleanup logic in afterEach hooks to ensure test isolation.
- **R-TEST-005** SHOULD: Keep test files focused on a single component or feature area to maintain clear test organization and avoid overly large test suites.
- **R-TEST-006** SHOULD: Limit describe nesting to 3 levels maximum and require clear naming at each level to avoid overly complex nested structures.

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
- Describe nesting does not exceed 3 levels without explicit justification

<enforcement>
Clause Code MUST NOT skip or defer verification. All new test files and modifications to existing test files must comply with these rules before merge. Violations are flagged during code review and developers must refactor tests to use the describe/it pattern. Exceptions require explicit justification in code comments and approval by tech lead or senior engineer.
</enforcement>