# Standardize Test Framework Structure with describe/it Blocks and Core Library Imports: Test Files Use

These rules are ALWAYS ACTIVE for all test files in the `tests/` directory, including unit, integration, and CLI test suites that validate agent implementations and configuration file handling.

### Rules

- **R-TEST-001** MUST: Organize test files using describe/it block structure for hierarchical test organization.
- **R-TEST-002** MUST: Import core Node.js libraries ('fs/promises', 'path', 'os') when performing filesystem operations in tests.
- **R-TEST-003** MUST: Use JSON.parse for validation of JSON configuration files (mcp.json, ruler.toml, settings.json) in test assertions.
- **R-TEST-004** MUST: Import and use shared test harness utilities from './harness' or '../tests/harness' for integration tests requiring project setup and teardown.
- **R-TEST-005** MAY: Test files MAY use child_process imports when testing CLI command execution with inherited stdio.
- **R-TEST-006** SHOULD: Call setupTestProject in beforeEach hooks and teardownTestProject in afterEach hooks to ensure test isolation and cleanup.
- **R-TEST-007** SHOULD: Use JSON.stringify with null and 2 parameters for consistent formatting when creating test fixture JSON files.
- **R-TEST-008** SHOULD: Organize describe blocks by component name (e.g., 'GooseAgent', 'CLI nested toggle precedence') with nested it blocks for specific behaviors.

### Verify

```bash
# Count describe blocks in test files
grep -r "describe(" tests/ | wc -l

# Count core library imports in test files
grep -r "from 'fs/promises'\|from 'path'\|from 'os'" tests/ | wc -l

# Count JSON.parse usage in test files
grep -r "JSON.parse" tests/ | wc -l

# Count test files using shared harness utilities
find tests/ -name '*.test.ts' -exec grep -l "./harness\|../tests/harness" {} \; | wc -l
```

**Accept when:**
- All test files in tests/ directory use describe/it block structure for test organization
- Test files performing filesystem operations import at least one of 'fs/promises', 'path', or 'os'
- Test files validating JSON configuration use JSON.parse for content verification
- Integration tests use shared test harness utilities for project setup and teardown
- setupTestProject and teardownTestProject are called in appropriate test lifecycle hooks
- describe blocks are organized by component name with nested it blocks for specific behaviors

<enforcement>
Clause Code MUST NOT skip or defer verification of test file structure compliance. All test files must conform to describe/it organization, required library imports, and harness utility usage before merge.
</enforcement>