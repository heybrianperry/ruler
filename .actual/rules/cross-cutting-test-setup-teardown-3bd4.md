# Standardize on describe/it Test Framework Pattern for Test Organization: Test Setup Teardown

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory, including unit, integration, and functional tests across all domains (agents, CLI, configuration, MCP integration, skills processing).

### Rules

- **R-TEST-001** MUST: Test setup and teardown logic MUST use `beforeEach` and `afterEach` lifecycle hooks when state needs to be initialized or cleaned up between tests.
- **R-TEST-002** MUST: Use `describe` blocks to name the component or feature under test, with nested `describe` blocks for specific aspects or scenarios.
- **R-TEST-003** MUST: Write `it` block descriptions as complete sentences starting with 'should' to clearly express expected behavior (e.g., 'should return error when input is invalid').
- **R-TEST-004** SHOULD: Keep test files focused on a single component or feature area to maintain clear test organization and avoid overly large test suites.
- **R-TEST-005** SHOULD: Limit `describe` nesting to 3 levels maximum to prevent overly complex nested structures that obscure test intent.
- **R-TEST-006** MAY: Document exceptions to the describe/it pattern in test file headers with explicit rationale and expected duration when the standard pattern cannot be used.

### Verify

```bash
# Count describe blocks in test files
grep -r "describe(" tests/ | wc -l

# Count it blocks in test files
grep -r "it(" tests/ | wc -l

# Count beforeEach/afterEach hooks in test files
grep -r "beforeEach\|afterEach" tests/ | wc -l
```

**Accept when:**
- All test files in `tests/` directory use `describe` blocks for test suite organization
- Individual test cases are defined using `it` blocks with descriptive behavior-focused names
- Test files requiring setup/teardown use `beforeEach`/`afterEach` hooks appropriately
- No test files exist outside the describe/it pattern without documented exceptions

<enforcement>
Claude Code MUST NOT skip or defer verification. All new and modified test files MUST conform to these rules before acceptance. Code review process checks for consistent use of describe/it pattern. Linting rules detect test files that do not follow the describe/it structure. CI pipeline verifies all tests are properly organized and executable.
</enforcement>