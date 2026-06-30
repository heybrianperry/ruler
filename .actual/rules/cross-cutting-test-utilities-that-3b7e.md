# Adopt Async Function Naming Convention for Test Setup Utilities: Test Utilities That

These rules are ALWAYS ACTIVE for all test utility functions that perform file system operations, including setup and teardown utilities in test harness modules and test files using describe/it frameworks.

### Rules

- **R-ASYNC-001** MUST: All test utilities that perform file system operations SHALL be declared as async functions.
- **R-ASYNC-002** SHOULD: Test utilities that parse JSON from file system reads SHOULD combine the read and parse operations in a single async function.
- **R-ASYNC-003** MUST: Test utilities MUST use descriptive verb-based names (read*, write*, create*, delete*, exists*, path*) that clearly indicate the I/O operation being performed.
- **R-ASYNC-004** MUST: Test utilities MUST import fs/promises module rather than callback-based fs module for all file system operations.
- **R-ASYNC-005** MUST: Test utilities MUST be exported from a centralized test harness module to promote reuse across test files.
- **R-ASYNC-006** MUST: All async utility calls in test files MUST be awaited to prevent unresolved promises and race conditions.

### Verify

```bash
# Count async functions with I/O operation names
grep -r "async function" tests/ | grep -E "(read|write|path|create|delete)" | wc -l

# Verify fs/promises imports in test files
grep -r "from 'fs/promises'" tests/ | wc -l

# Verify await usage with async utilities
grep -r "await.*\(read\|write\|path\)" tests/ | wc -l

# Check for floating promises (ESLint)
eslint tests/ --rule '@typescript-eslint/no-floating-promises: error'

# Check for missing await on Promise-returning functions
eslint tests/ --rule '@typescript-eslint/require-await: error'
```

**Accept when:**
- All test utilities that perform file system operations are declared as async functions
- Test files consistently import fs/promises module and await all async utility calls
- Async test utilities use verb-based naming conventions that clearly indicate I/O operations
- ESLint rules (@typescript-eslint/no-floating-promises, @typescript-eslint/require-await) pass without violations
- TypeScript compiler in strict mode compiles without errors related to missing await on Promise-returning functions
- All file system operations in test utilities use fs/promises module, not callback-based fs

<enforcement>
Claude Code MUST NOT skip or defer verification. All async function naming conventions and await patterns MUST be verified before accepting test utility code. ESLint and TypeScript compiler checks are mandatory in the CI pipeline.
</enforcement>