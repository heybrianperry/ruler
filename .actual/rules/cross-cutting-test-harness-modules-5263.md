# Adopt Async Function Naming Convention for Test Setup Utilities: Test Harness Modules

These rules are ALWAYS ACTIVE for all test harness modules and test setup utilities that perform file system operations.

### Rules

- **R-HARNESS-001** MUST: All test harness modules MUST export reusable async utilities for common file system operations to promote consistency across test files.
- **R-HARNESS-002** MUST: Test utilities that perform file system operations MUST be declared as async functions.
- **R-HARNESS-003** MUST: Test utilities MUST import and use the fs/promises module rather than callback-based fs module for all file system operations.
- **R-HARNESS-004** SHOULD: Test utilities SHOULD use descriptive verb-based names (read*, write*, create*, delete*, exists*) that clearly indicate the I/O operation being performed.
- **R-HARNESS-005** SHOULD: Related operations (e.g., readFile + JSON.parse) SHOULD be combined into single async utilities to reduce boilerplate in test files.
- **R-HARNESS-006** MUST: All async utility calls in test files MUST be awaited to prevent unresolved promises and race conditions.
- **R-HARNESS-007** MUST: Test harness modules MUST establish standard error handling patterns for async utilities and ensure all async functions properly propagate errors.

### Verify

```bash
# Count async functions in test utilities with I/O operation names
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
- Related file system operations are combined into single async utilities rather than inlined
- Error handling is consistent across all async test utilities

<enforcement>
Clause Code MUST NOT skip or defer verification. ESLint rules (@typescript-eslint/no-floating-promises, @typescript-eslint/require-await) are enforced in CI pipeline. TypeScript compiler strict mode catches missing await on Promise-returning functions. Code review blocks merge if async test utilities do not follow naming conventions.
</enforcement>