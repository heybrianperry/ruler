# Standardize Test Framework Structure with describe/it Blocks and Core Library Imports: Test Files Use

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory, including unit, integration, and CLI test suites that validate agent implementations and configuration file handling.

### Rules

- **R-TEST-001** MUST: Test files MUST use describe/it block structure for organizing test suites and individual test cases.
- **R-TEST-002** MUST: Test files performing filesystem operations MUST import at least one of 'fs/promises', 'path', or 'os' from Node.js core libraries.
- **R-TEST-003** MUST: Test files validating JSON configuration MUST use JSON.parse for content verification.
- **R-TEST-004** MUST: Integration tests MUST use shared test harness utilities (imported from './harness' or '../tests/harness') for project setup and teardown.
- **R-TEST-005** SHOULD: Test harness utilities SHOULD be called via setupTestProject with file structure object to create isolated test environments.
- **R-TEST-006** SHOULD: Teardown operations SHOULD be called in afterEach hooks to ensure cleanup even when tests fail.
- **R-TEST-007** SHOULD: Describe blocks SHOULD be organized by component name and nest it blocks for specific behaviors.
- **R-TEST-008** SHOULD: JSON.stringify operations SHOULD use null and 2 parameters for consistent formatting when creating test fixture JSON files.

### Verify

```bash
# Count describe blocks in test files
grep -r "describe(" tests/ | wc -l

# Count core library imports in test files
grep -r "from 'fs/promises'\|from 'path'\|from 'os'" tests/ | wc -l

# Count JSON.parse operations in test files
grep -r "JSON.parse" tests/ | wc -l

# Count test files using shared test harness utilities
find tests/ -name '*.test.ts' -exec grep -l "./harness\|../tests/harness" {} \; | wc -l
```

**Accept when:**
- All test files in tests/ directory use describe/it block structure for test organization
- Test files performing filesystem operations import at least one of 'fs/promises', 'path', or 'os'
- Test files validating JSON configuration use JSON.parse for content verification
- Integration tests use shared test harness utilities for project setup and teardown
- Describe blocks are organized by component name with nested it blocks for specific behaviors
- Test harness utilities are imported from './harness' or '../tests/harness' depending on file location

<enforcement>
Clause Code MUST NOT skip or defer verification of test file structure compliance. CI pipeline MUST fail if test files lack describe/it structure or required core library imports. Code review MUST request restructuring of non-compliant test files. Exception process requires documentation of justification in test file header comment and approval from testing infrastructure maintainer.
</enforcement>