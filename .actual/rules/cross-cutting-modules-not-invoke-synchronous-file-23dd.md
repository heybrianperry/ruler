# fs/promises Module for Asynchronous File System Operations: Modules Not Invoke Synchronous File System

These rules are ALWAYS ACTIVE for all application modules that read from, write to, or traverse disk file systems, including command-line handlers, model context protocol synchronizers, and autonomous agent persistence layers.

### Rules

- **R-FS-001** MUST_NOT: Modules MUST_NOT invoke synchronous file system methods on the main execution thread.

### Verify

```bash
# Discover the repository test runner script from the project manifest and execute the test suite validating file operations and asynchronous handler execution.
# Discover the static analysis and linting script from the project manifest and run it to verify that no synchronous file system APIs are invoked across the codebase.
```

**Accept when:**
- All asynchronous file system operations execute without unhandled promise rejections.
- Automated test suites pass with zero regressions across file-dependent modules.
- Static analysis confirms zero invocations of blocking synchronous file system functions.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>