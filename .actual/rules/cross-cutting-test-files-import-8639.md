# Standardize Test Framework Structure with describe/it Blocks and Core Library Imports: Test Files Import

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory, including unit, integration, and CLI test suites that validate agent implementations and configuration file handling.

### Rules

- **R-TEST-001** MUST: Test files MUST import core Node.js libraries ('fs/promises', 'path', 'os') when performing filesystem or environment operations.
- **R-TEST-002** MUST: Test files MUST use describe/it block structure for test organization.
- **R-TEST-003** MUST: Integration tests MUST use shared test harness utilities (setupTestProject/teardownTestProject) for project setup and teardown.
- **R-TEST-004** SHOULD: Test files validating JSON configuration SHOULD use JSON.parse for content verification.
- **R-TEST-005** SHOULD: Organize describe blocks by component name and nest it blocks for specific behaviors.
- **R-TEST-006** SHOULD: Use JSON.stringify with null and 2 parameters for consistent formatting when creating test fixture JSON files.
- **R-TEST-007** SHOULD: Call teardownTestProject in afterEach hooks to ensure cleanup even when tests fail.

### Verify

```bash
# Verify describe/it block usage
grep -r "describe(" tests/ | wc -l

# Verify core library imports
grep -r "from 'fs/promises'\|from 'path'\|from 'os'" tests/ | wc -l

# Verify JSON.parse usage in configuration tests
grep -r "JSON.parse" tests/ | wc -l

# Verify shared test harness imports
find tests/ -name '*.test.ts' -exec grep -l "./harness\|../tests/harness" {} \; | wc -l
```

**Accept when:**
- All test files in tests/ directory use describe/it block structure for test organization
- Test files performing filesystem operations import at least one of 'fs/promises', 'path', or 'os'
- Test files validating JSON configuration use JSON.parse for content verification
- Integration tests use shared test harness utilities for project setup and teardown
- Test harness utilities are imported from './harness' or '../tests/harness' depending on test file location

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. All test files must comply with R-TEST-001 through R-TEST-007 before merge.
</enforcement>