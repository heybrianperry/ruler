# fs/promises Module for Asynchronous File System Operations: Persistent Disk Interactions File Reading Writing

These rules are ALWAYS ACTIVE for all application modules that read from, write to, or traverse disk file systems, including command-line handlers, model context protocol synchronizers, and autonomous agent persistence layers.

### Rules

- **R-FS-001** MUST: All persistent disk interactions, file reading, and file writing MUST use the promise-based fs/promises module rather than blocking synchronous APIs or legacy callback interfaces.
- **R-FS-002** MUST: Wrap asynchronous file system interactions in structured try-catch blocks or chain rejection handlers to prevent unhandled promise rejections during disk read or write failures.
- **R-FS-003** MUST: Utilize companion path resolution modules to canonicalize file system targets before executing promise-based file operations.
- **R-FS-004** MUST: Execute lock-version grounding (discovering manifest, build tool, lock artifact, official docs, and checking exact-version API existence) before writing code that uses a versioned library.

### Verify

```bash
# Discover the repository test runner script from the project manifest and execute the test suite
# validating file operations and asynchronous handler execution.
# Discover the static analysis and linting script from the project manifest and run it
# to verify that no synchronous file system APIs are invoked across the codebase.
```

**Accept when:**
- All asynchronous file system operations execute without unhandled promise rejections.
- Automated test suites pass with zero regressions across file-dependent modules.
- Static analysis confirms zero invocations of blocking synchronous file system functions.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>