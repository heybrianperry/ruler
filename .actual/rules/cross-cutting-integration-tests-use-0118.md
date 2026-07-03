# Standardize Test Framework Structure with describe/it Blocks and Core Library Imports: Integration Tests Use

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory, including unit, integration, and CLI test suites that validate agent implementations and configuration file handling.

### Rules

- **R-TEST-001** SHOULD: Integration tests SHOULD use beforeEach/afterEach hooks with setupTestProject and teardownTestProject patterns for test isolation.
- **R-TEST-002** MUST: All test files in tests/ directory MUST use describe/it block structure for test organization.
- **R-TEST-003** MUST: Test files performing filesystem operations MUST import at least one of 'fs/promises', 'path', or 'os'.
- **R-TEST-004** MUST: Test files validating JSON configuration MUST use JSON.parse for content verification.
- **R-TEST-005** MUST: Integration tests MUST use shared test harness utilities imported from './harness' or '../tests/harness' for project setup and teardown.
- **R-TEST-006** SHOULD: Organize describe blocks by component name and nest it blocks for specific behaviors.
- **R-TEST-007** SHOULD: Use JSON.stringify with null and 2 parameters for consistent formatting when creating test fixture JSON files.
- **R-TEST-008** SHOULD: Use path.join and os.tmpdir() consistently to avoid platform-specific filesystem behavior.

### Verify

```bash
# Verify describe/it block usage across test files
grep -r "describe(" tests/ | wc -l

# Verify core library imports for filesystem operations
grep -r "from 'fs/promises'\|from 'path'\|from 'os'" tests/ | wc -l

# Verify JSON.parse usage for configuration validation
grep -r "JSON.parse" tests/ | wc -l

# Verify shared test harness imports in integration tests
find tests/ -name '*.test.ts' -exec grep -l "./harness\|../tests/harness" {} \; | wc -l
```

**Accept when:**
- All test files in tests/ directory use describe/it block structure for test organization
- Test files performing filesystem operations import at least one of 'fs/promises', 'path', or 'os'
- Test files validating JSON configuration use JSON.parse for content verification
- Integration tests use shared test harness utilities for project setup and teardown
- beforeEach/afterEach hooks are present in integration tests with setupTestProject and teardownTestProject calls
- describe blocks are organized by component name with nested it blocks for specific behaviors

<enforcement>
Clause Code MUST NOT skip or defer verification of test structure compliance. CI pipeline MUST fail if test files lack describe/it structure, required core library imports, or shared harness utilities. Code review MUST verify test structure compliance against this ruleset.
</enforcement>