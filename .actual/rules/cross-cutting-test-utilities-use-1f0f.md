# Adopt Async Function Naming Convention for Test Setup Utilities: Test Utilities Use

These rules are ALWAYS ACTIVE for all test utility functions and test setup code that perform file system operations.

### Rules

- **R-TEST-ASYNC-001** MUST: All test utilities that perform file system operations (read, write, create, delete, exists checks) SHALL be declared as async functions.
- **R-TEST-ASYNC-002** MUST: Test files SHALL import `fs/promises` module rather than callback-based `fs` module for all file system operations in test utilities.
- **R-TEST-ASYNC-003** MUST: All invocations of async test utilities in test code SHALL use the `await` keyword to ensure proper promise resolution.
- **R-TEST-ASYNC-004** SHOULD: Test utilities SHOULD use descriptive verb-based names (read*, write*, create*, delete*, exists*, path*) that clearly indicate the I/O operation being performed.
- **R-TEST-ASYNC-005** SHOULD: Related file system operations (e.g., readFile + JSON.parse) SHOULD be combined into single async utilities to reduce boilerplate in test files.
- **R-TEST-ASYNC-006** MAY: Test utilities MAY use nested async operations when setting up complex test fixtures involving multiple file system operations.
- **R-TEST-ASYNC-007** SHOULD: Independent file system operations in test setup SHOULD be parallelized using Promise.all() where possible to avoid performance bottlenecks.
- **R-TEST-ASYNC-008** SHOULD: Async test utilities SHOULD be exported from a centralized test harness module (e.g., tests/harness) to promote reuse across test files.

### Verify

```bash
# Count async functions with I/O operation names in test utilities
grep -r "async function" tests/ | grep -E "(read|write|path|create|delete)" | wc -l

# Verify fs/promises module is imported in test files
grep -r "from 'fs/promises'" tests/ | wc -l

# Verify await is used with async utility calls
grep -r "await.*\(read\|write\|path\)" tests/ | wc -l

# Check for floating promises (missing await) using ESLint
eslint --rule '@typescript-eslint/no-floating-promises: error' tests/

# Verify async functions have proper return type annotations
grep -r "async function.*:.*Promise" tests/ | wc -l
```

**Accept when:**
- All test utilities that perform file system operations are declared as async functions
- Test files consistently import `fs/promises` module and await all async utility calls
- Async test utilities use verb-based naming conventions that clearly indicate I/O operations
- ESLint detects zero floating promises in test files
- All async test utilities are exported from centralized harness modules
- No synchronous file system operations (readFileSync, writeFileSync) are used in test utilities except where explicitly documented with justification

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for test utility code. Violations detected by ESLint (@typescript-eslint/no-floating-promises, @typescript-eslint/require-await) or TypeScript strict mode MUST be resolved before code review approval.
</enforcement>