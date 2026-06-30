# Standardize Node.js Core Module Usage for File System and Path Operations: Use Tmpdir Temporary

These rules are ALWAYS ACTIVE for all agent implementations, core processors, utility modules, test infrastructure, and configuration loaders that perform file system and path operations.

### Rules

- **R-TMPDIR-001** MUST: Use `fs/promises` for all async file system operations; never use synchronous `fs` methods (`readFileSync`, `writeFileSync`, etc.) in src/agents/ or src/core/ directories.
- **R-TMPDIR-002** MUST: Use `path.join()` or `path.resolve()` for all file path construction; never concatenate paths using string concatenation with `/` or `\`.
- **R-TMPDIR-003** SHOULD: Use `os.tmpdir()` for temporary directory operations in test infrastructure and temporary file creation.
- **R-TMPDIR-004** MUST: Ensure cleanup of temporary directories in test infrastructure using `fs.rm(path, { recursive: true, force: true })` in afterEach/afterAll hooks.
- **R-TMPDIR-005** MUST: Wrap `JSON.parse()` in try-catch blocks when parsing configuration files read via `fs/promises`.
- **R-TMPDIR-006** SHOULD: Use `fs.access()` with `.then(() => true).catch(() => false)` pattern for file existence checks rather than deprecated `fs.exists()`.
- **R-TMPDIR-007** MUST: Import fs/promises using `import fs from "fs/promises"` or named imports for tree-shaking benefits.
- **R-TMPDIR-008** MUST NOT: Introduce third-party file system libraries (fs-extra, graceful-fs) for core functionality without architectural review.

### Verify

```bash
# Check for synchronous fs methods in core directories
grep -r "readFileSync\|writeFileSync" src/agents/ src/core/ --include='*.ts' | wc -l | grep -q '^0$'

# Verify fs/promises is used instead of require('fs')
grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v "fs/promises" | wc -l | grep -q '^0$'

# Check path module usage
grep -r "from ['\"]path['\"]" src/ tests/ --include='*.ts' | wc -l

# Verify no third-party fs libraries in dependencies
npm ls fs-extra graceful-fs 2>&1 | grep -q 'empty' || echo 'Third-party fs libs detected'
```

**Accept when:**
- All source files in src/agents/ and src/core/ use `fs/promises` for async file operations with zero synchronous fs method calls
- All path construction uses `path.join()` or `path.resolve()` with no string concatenation for file paths
- No third-party file system libraries (fs-extra, graceful-fs) appear in package.json dependencies for core functionality
- Test infrastructure consistently uses `os.tmpdir()` for temporary directories and `fs.rm()` with force option for cleanup
- All JSON parsing from file reads is wrapped in try-catch blocks
- File existence checks use the `.then()/.catch()` pattern rather than deprecated methods

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via grep patterns in CI pipeline checking for synchronous fs methods and string-based path concatenation is mandatory. Code review checklist must verify fs/promises and path module usage. Dependency audit in CI must reject PRs introducing third-party file system libraries without architectural review. CI build fails if synchronous fs methods detected in src/agents/ or src/core/ directories. PR review blocks merge if path operations use string concatenation instead of path module methods.
</enforcement>