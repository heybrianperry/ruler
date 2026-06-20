# Standardize Test Framework Structure with describe/it Blocks and Core Library Imports: Test Files Import

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory, including unit, integration, and CLI test suites that validate agent implementations and configuration file handling.

### Rules

- **R-TEST-001** SHOULD: Test files SHOULD import shared test harness utilities from relative paths ('./harness', '../tests/harness') for common setup and teardown operations.
- **R-TEST-002** SHOULD: Test files SHOULD use describe/it block structure for organizing test cases hierarchically by component name and specific behaviors.
- **R-TEST-003** SHOULD: Test files performing filesystem operations SHOULD import at least one of 'fs/promises', 'path', or 'os' from Node.js core libraries.
- **R-TEST-004** SHOULD: Test files validating JSON configuration SHOULD use JSON.parse for content verification with try-catch error handling.
- **R-TEST-005** SHOULD: Integration tests SHOULD call setupTestProject with file structure objects to create isolated test environments and teardownTestProject in afterEach hooks.

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
- All test files in tests/ directory use describe/it block structure for test organization
- Test files performing filesystem operations import at least one of 'fs/promises', 'path', or 'os'
- Test files validating JSON configuration use JSON.parse for content verification
- Integration tests use shared test harness utilities for project setup and teardown
- Test harness imports use relative paths ('./harness' or '../tests/harness') appropriate to test file location

<enforcement>
Clause Code MUST NOT skip or defer verification of test file structure compliance. Violations detected by CI pipeline grep-based verification or code review checklist MUST be addressed before merge.
</enforcement>