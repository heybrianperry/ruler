# Standardize Test Framework Structure with describe/it Blocks and Core Library Imports: Test Files Validating

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory, including unit, integration, and CLI test suites validating agent implementations and configuration file handling.

### Rules

- **R-TEST-001** MUST: Test files validating JSON configuration MUST use JSON.parse for input validation and content verification.
- **R-TEST-002** MUST: All test files in tests/ directory MUST use describe/it block structure for test organization.
- **R-TEST-003** MUST: Test files performing filesystem operations MUST import at least one of 'fs/promises', 'path', or 'os'.
- **R-TEST-004** MUST: Integration tests MUST use shared test harness utilities from './harness' or '../tests/harness' for project setup and teardown.
- **R-TEST-005** SHOULD: Organize describe blocks by component name and nest it blocks for specific behaviors.
- **R-TEST-006** SHOULD: Use setupTestProject with file structure object to create isolated test environments with required configuration files.
- **R-TEST-007** SHOULD: Call teardownTestProject in afterEach hooks to ensure cleanup even when tests fail.
- **R-TEST-008** SHOULD: Wrap JSON.parse in try-catch blocks with descriptive error context for malformed test fixtures.

### Verify

```bash
# Verify describe/it block usage across test files
grep -r "describe(" tests/ | wc -l

# Verify core library imports for filesystem operations
grep -r "from 'fs/promises'\|from 'path'\|from 'os'" tests/ | wc -l

# Verify JSON.parse usage in configuration validation tests
grep -r "JSON.parse" tests/ | wc -l

# Verify shared test harness imports in integration tests
find tests/ -name '*.test.ts' -exec grep -l "./harness\|../tests/harness" {} \; | wc -l
```

**Accept when:**
- All test files in tests/ directory use describe/it block structure for test organization
- Test files performing filesystem operations import at least one of 'fs/promises', 'path', or 'os'
- Test files validating JSON configuration use JSON.parse for content verification
- Integration tests use shared test harness utilities for project setup and teardown
- JSON.parse operations are wrapped in try-catch blocks with descriptive error context
- describe blocks are organized by component name with nested it blocks for specific behaviors

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. Violations detected by CI pipeline grep-based verification or code review checklist MUST be addressed before merge. Test files lacking describe/it structure, required core library imports, or shared harness utilities MUST be restructured or documented with ADR-AUTO-EXCEPTION markers.
</enforcement>