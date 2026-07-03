# Adopt Async Function Naming Convention for Test Setup Utilities: Async Test Utilities

These rules are ALWAYS ACTIVE for all test setup utilities and harness functions that perform file system operations across the test suite.

### Rules

- **R-ASYNC-001** MUST: Async test utilities MUST use verb-based naming that clearly indicates the I/O operation (e.g., readGeminiSettings, writeProject, writeAgent, pathExists).
- **R-ASYNC-002** MUST: All test utilities that perform file system operations MUST be declared as async functions.
- **R-ASYNC-003** MUST: Test files MUST import fs/promises module rather than callback-based fs module for all file system operations in test utilities.
- **R-ASYNC-004** MUST: All async utility calls in test files MUST be awaited to prevent unresolved promises and race conditions.
- **R-ASYNC-005** SHOULD: Related operations (e.g., readFile + JSON.parse) SHOULD be combined into single async utilities to reduce boilerplate in test files.
- **R-ASYNC-006** SHOULD: Async test utilities SHOULD be exported from a centralized test harness module to promote reuse across test files.
- **R-ASYNC-007** SHOULD: Descriptive verb-based names (read*, write*, create*, delete*, exists*) SHOULD be used to clearly indicate the I/O operation being performed.

### Verify

```bash
# Count async functions with I/O operation naming patterns
grep -r "async function" tests/ | grep -E "(read|write|path|create|delete)" | wc -l

# Verify fs/promises module is imported in test files
grep -r "from 'fs/promises'" tests/ | wc -l

# Verify await statements are used with async utilities
grep -r "await.*\(read\|write\|path\)" tests/ | wc -l
```

**Accept when:**
- All test utilities that perform file system operations are declared as async functions
- Test files consistently import fs/promises module and await all async utility calls
- Async test utilities use verb-based naming conventions that clearly indicate I/O operations
- ESLint rules (@typescript-eslint/no-floating-promises, @typescript-eslint/require-await) pass without violations
- TypeScript compiler strict mode compiles without errors related to missing await on Promise-returning functions

<enforcement>
Claude Code MUST NOT skip or defer verification. ESLint and TypeScript compiler checks are mandatory in CI pipeline. Code review MUST verify async/await patterns and naming conventions before merge approval.
</enforcement>