# Standardize Test Framework Structure with describe/it Blocks and Core Library Imports: Test Files Organize

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory, including unit, integration, and CLI test suites that validate agent implementations and configuration file handling.

### Rules

- **R-TEST-001** SHOULD: Test files SHOULD organize related test cases under descriptive describe blocks that indicate the component or feature under test.
- **R-TEST-002** SHOULD: Test files performing filesystem operations SHOULD import at least one of `fs/promises`, `path`, or `os` from Node.js core libraries.
- **R-TEST-003** SHOULD: Test files validating JSON configuration SHOULD use `JSON.parse` for content verification.
- **R-TEST-004** SHOULD: Integration tests SHOULD use shared test harness utilities (imported from `./harness` or `../tests/harness`) for project setup and teardown.
- **R-TEST-005** SHOULD: Test harness utilities SHOULD be called with `setupTestProject` to create isolated test environments and `teardownTestProject` in `afterEach` hooks for cleanup.
- **R-TEST-006** SHOULD: Describe blocks SHOULD be organized by component name (e.g., 'GooseAgent', 'CLI nested toggle precedence') with nested `it` blocks for specific behaviors.
- **R-TEST-007** SHOULD: JSON fixture files SHOULD use `JSON.stringify` with `null` and `2` parameters for consistent formatting.

### Verify

```bash
# Count describe blocks in test files
grep -r "describe(" tests/ | wc -l

# Count core library imports
grep -r "from 'fs/promises'\|from 'path'\|from 'os'" tests/ | wc -l

# Count JSON.parse usage
grep -r "JSON.parse" tests/ | wc -l

# Count test files using shared harness utilities
find tests/ -name '*.test.ts' -exec grep -l "./harness\|../tests/harness" {} \; | wc -l
```

**Accept when:**
- All test files in `tests/` directory use `describe/it` block structure for test organization
- Test files performing filesystem operations import at least one of `fs/promises`, `path`, or `os`
- Test files validating JSON configuration use `JSON.parse` for content verification
- Integration tests use shared test harness utilities for project setup and teardown
- Describe blocks are organized by component name with nested `it` blocks for specific behaviors
- JSON fixture files use consistent formatting via `JSON.stringify(data, null, 2)`

<enforcement>
Clause Code MUST NOT skip or defer verification of test file structure compliance. Violations detected by CI pipeline grep-based verification or code review checklist MUST be addressed before merge.
</enforcement>