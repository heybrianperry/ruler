# Adopt Async Function Naming Convention for Test Setup Utilities: Test Files Await

These rules are ALWAYS ACTIVE for all test files that use async setup utilities for file system operations, including test harness modules and integration/unit test files that manage test fixtures, configuration files, and temporary file system state.

### Rules

- **R-ASYNC-001** MUST: Test files MUST await all async setup utilities before proceeding with test assertions.
- **R-ASYNC-002** MUST: All test utilities that perform file system operations MUST be declared as async functions.
- **R-ASYNC-003** MUST: Test files MUST import fs/promises module rather than callback-based fs module for all file system operations.
- **R-ASYNC-004** SHOULD: Test utilities SHOULD use descriptive verb-based names (read*, write*, create*, delete*, exists*) that clearly indicate the I/O operation being performed.
- **R-ASYNC-005** SHOULD: Related operations (e.g., readFile + JSON.parse) SHOULD be combined into single async utilities to reduce boilerplate in test files.
- **R-ASYNC-006** SHOULD: Independent file system operations in test setup SHOULD be parallelized using Promise.all() where possible to avoid performance bottlenecks.

### Verify

```bash
# Count async functions with I/O operation names in tests
grep -r "async function" tests/ | grep -E "(read|write|path|create|delete)" | wc -l

# Verify fs/promises module is imported
grep -r "from 'fs/promises'" tests/ | wc -l

# Verify await statements are used with async utilities
grep -r "await.*\(read\|write\|path\)" tests/ | wc -l

# Check for floating promises (should return 0)
grep -r "(read\|write\|path).*()" tests/ | grep -v "await" | wc -l
```

**Accept when:**
- All test utilities that perform file system operations are declared as async functions
- Test files consistently import fs/promises module and await all async utility calls
- Async test utilities use verb-based naming conventions that clearly indicate I/O operations
- ESLint rules (@typescript-eslint/no-floating-promises, @typescript-eslint/require-await) pass in CI pipeline
- TypeScript compiler strict mode reports no missing await on Promise-returning functions

<enforcement>
Clause Code MUST NOT skip or defer verification. ESLint rules are enforced in CI pipeline. Code review must verify async/await patterns in test utilities. TypeScript compiler strict mode must catch missing await statements. Violations cause CI build failure and block merge.
</enforcement>