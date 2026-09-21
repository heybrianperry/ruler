# fs/promises Module for Asynchronous File System Operations: Developers Inspect Repository Lock Artifact Verify

These rules are ALWAYS ACTIVE for all application modules that read from, write to, or traverse disk file systems, including command-line handlers, model context protocol synchronizers, and autonomous agent persistence layers.

### Rules

- **R-FSP-001** MUST: Developers MUST inspect the repository lock artifact to verify the exact resolved version of any versioned runtime dependencies or typing packages before implementing or modifying file system interfaces.
- **R-FSP-002** MUST: Wrap asynchronous file system interactions in structured try-catch blocks or chain rejection handlers to prevent unhandled promise rejections during disk read or write failures.
- **R-FSP-003** MUST: Utilize companion path resolution modules to canonicalize file system targets before executing promise-based file operations.

### Verify

```bash
# Discover and run the static analysis and linting script from the project manifest to verify no synchronous file system APIs are invoked
# Discover and run the project test runner script validating file operations and asynchronous handler execution
```

**Accept when:**
- All asynchronous file system operations execute without unhandled promise rejections.
- Automated test suites pass with zero regressions across file-dependent modules.
- Static analysis confirms zero invocations of blocking synchronous file system functions.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests containing synchronous file system invocations or unhandled file promise rejections must be blocked from merging.
</enforcement>