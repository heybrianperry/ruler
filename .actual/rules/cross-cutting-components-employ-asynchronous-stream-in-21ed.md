# fs/promises Module for Asynchronous File System Operations: Components Employ Asynchronous Stream Interfaces When

These rules are ALWAYS ACTIVE for all application modules that read from, write to, or traverse disk file systems, including command-line handlers, model context protocol synchronizers, and autonomous agent persistence layers.

### Rules

- **R-FS-001** MAY: Components MAY employ asynchronous stream interfaces when streaming file contents that exceed standard memory buffer limits.
- **R-FS-002** MANDATORY: Wrap asynchronous file system interactions in structured try-catch blocks or chain rejection handlers to prevent unhandled promise rejections during disk read or write failures.
- **R-FS-003** MANDATORY: Utilize companion path resolution modules to canonicalize file system targets before executing promise-based file operations.
- **R-FS-004** MANDATORY: Before writing code that uses a versioned library, find the dependency manifest, identify the build tool, inspect the repository lock or resolution artifact for the exact resolved version, and confirm every API, class, or function exists in that version's documentation.

### Verify

```bash
# Discover the repository test runner script from the project manifest and execute the test suite
# validating file operations and asynchronous handler execution.
# Discover the static analysis and linting script from the project manifest and run it to verify
# that no synchronous file system APIs are invoked across the codebase.
```

**Accept when:**
- All asynchronous file system operations execute without unhandled promise rejections.
- Automated test suites pass with zero regressions across file-dependent modules.
- Static analysis confirms zero invocations of blocking synchronous file system functions.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated continuous integration pipelines and peer code reviews. Violations block pull requests from merging and must be refactored to use asynchronous fs/promises methods.
</enforcement>