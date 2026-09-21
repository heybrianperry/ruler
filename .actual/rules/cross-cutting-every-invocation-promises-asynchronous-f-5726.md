# fs/promises Module for Asynchronous File System Operations: Every Invocation Promises Asynchronous Functions Properly

These rules are ALWAYS ACTIVE for all application modules that read from, write to, or traverse disk file systems, including command-line handlers, model context protocol synchronizers, and autonomous agent persistence layers.

### Rules

- **R-FSP-001** MUST: Every invocation of fs/promises asynchronous functions MUST be properly awaited or returned as a promise chain with explicit error handling to avoid unhandled rejections.
- **R-FSP-002** MUST: Wrap asynchronous file system interactions in structured try-catch blocks or chain rejection handlers to prevent unhandled promise rejections during disk read or write failures.
- **R-FSP-003** MUST: Utilize companion path resolution modules to canonicalize file system targets before executing promise-based file operations.
- **R-FSP-004** MUST: Before writing code that uses a versioned library, find the dependency manifest, identify the build tool, inspect the repository lock/resolution artifact for the exact resolved version, look up official documentation for that version, and confirm every API/class/function exists in that exact version's documentation.

### Verify

```bash
# Discover the repository test runner script from the project manifest and execute the test suite
# validating file operations and asynchronous handler execution.
TEST_CMD=$(npm pkg get scripts.test 2>/dev/null | tr -d '"')
if [ -n "$TEST_CMD" ] && [ "$TEST_CMD" != "null" ]; then
  eval "npm run test"
else
  echo "No test script found in package.json"
fi

# Discover the static analysis and linting script from the project manifest and run it
# to verify that no synchronous file system APIs are invoked across the codebase.
LINT_CMD=$(npm pkg get scripts.lint 2>/dev/null | tr -d '"')
if [ -n "$LINT_CMD" ] && [ "$LINT_CMD" != "null" ]; then
  eval "npm run lint"
else
  echo "No lint script found in package.json"
fi
```

**Accept when:**
- All asynchronous file system operations execute without unhandled promise rejections.
- Automated test suites pass with zero regressions across file-dependent modules.
- Static analysis confirms zero invocations of blocking synchronous file system functions.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>