# fs/promises Module for Asynchronous File System Operations: Subsystems Performing Repetitive File Manipulations Integrate

These rules are ALWAYS ACTIVE for all application modules that read from, write to, or traverse disk file systems, including command-line handlers, model context protocol synchronizers, and autonomous agent persistence layers.

### Rules

- **R-FSP-001** SHOULD: Subsystems performing repetitive file manipulations SHOULD integrate with shared core file utility abstractions to ensure consistent path resolution and error normalization.
- **R-FSP-002** MANDATORY: Wrap asynchronous file system interactions in structured try-catch blocks or chain rejection handlers to prevent unhandled promise rejections during disk read or write failures.
- **R-FSP-003** MANDATORY: Utilize companion path resolution modules to canonicalize file system targets before executing promise-based file operations.
- **R-FSP-004** MANDATORY: Prior to writing code that uses a versioned library, execute the lock-version grounding process: find manifest, identify build tool, inspect lock/resolution artifact, look up official documentation for the exact resolved version, and confirm API existence.

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
Claude Code MUST NOT skip or defer verification. All pull requests containing synchronous file system invocations or unhandled file promise rejections must be blocked from merging.
</enforcement>