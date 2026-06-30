# Adopt Async Function Naming Convention for Test Setup Utilities: Test Setup Utilities

These rules are ALWAYS ACTIVE for all test setup utilities and harness functions that perform file system operations (read, write, exists checks) across the test suite.

### Rules

- **R-ASYNC-001** MUST: Test setup utilities that perform file system operations (read, write, exists checks) MUST be implemented as async functions.
- **R-ASYNC-002** MUST: Import fs/promises module rather than callback-based fs module for all test utilities that perform file system operations.
- **R-ASYNC-003** MUST: Export async test utilities from a centralized test harness module (e.g., tests/harness) to promote reuse across test files.
- **R-ASYNC-004** SHOULD: Use descriptive verb-based names (read*, write*, create*, delete*, exists*) that clearly indicate the I/O operation being performed.
- **R-ASYNC-005** SHOULD: Combine related operations (e.g., readFile + JSON.parse) into single async utilities to reduce boilerplate in test files.
- **R-ASYNC-006** MUST: All async utility calls in test files MUST be awaited to prevent unresolved promises and race conditions.

### Verify

```bash
# Count async functions in test utilities matching I/O operation patterns
grep -r "async function" tests/ | grep -E "(read|write|path|create|delete)" | wc -l

# Verify fs/promises module is imported in test files
grep -r "from 'fs/promises'" tests/ | wc -l

# Verify async utility calls are awaited
grep -r "await.*\(read\|write\|path\)" tests/ | wc -l
```

**Accept when:**
- All test utilities that perform file system operations are declared as async functions
- Test files consistently import fs/promises module and await all async utility calls
- Async test utilities use verb-based naming conventions that clearly indicate I/O operations
- ESLint rules (@typescript-eslint/no-floating-promises, @typescript-eslint/require-await) pass in CI pipeline
- TypeScript compiler strict mode reports no missing await on Promise-returning functions

<enforcement>
Claude Code MUST NOT skip or defer verification. ESLint and TypeScript compiler checks are mandatory in CI. Code review MUST verify async/await patterns before merge. Violations block the build.
</enforcement>