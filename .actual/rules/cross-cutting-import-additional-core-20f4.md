# Standardize Node.js Core Module Usage for File System and Path Operations: Import Additional Core

These rules are ALWAYS ACTIVE for all agent implementations, core processors, utility modules, test infrastructure, and configuration loaders that perform file system and path operations.

### Rules

- **R-CORE-001** MUST: Import `fs/promises` for all async file system operations; never use synchronous `fs` methods (`readFileSync`, `writeFileSync`, etc.) in src/agents/ or src/core/ directories.
- **R-CORE-002** MUST: Use `path.join()` or `path.resolve()` for all file path construction; never concatenate paths using string concatenation with `/` or `\`.
- **R-CORE-003** MUST: Wrap `JSON.parse()` in try-catch blocks when parsing configuration files to handle malformed JSON gracefully.
- **R-CORE-004** MUST: Use `fs.access()` with `.then(() => true).catch(() => false)` pattern for file existence checks instead of deprecated `fs.exists()`.
- **R-CORE-005** MUST: Ensure test infrastructure cleanup using `fs.rm(path, { recursive: true, force: true })` in afterEach/afterAll hooks to prevent test pollution.
- **R-CORE-006** MAY: Import additional core modules (`js-yaml`, `@iarna/toml`) for structured data parsing when required by configuration formats.
- **R-CORE-007** SHOULD: Use `os.tmpdir()` for temporary directory creation in test infrastructure and cleanup operations.
- **R-CORE-008** SHOULD: Implement path validation using `isPathInsideOrEqual` utility or equivalent sanitization for all user-supplied paths before `path.join()` or `path.resolve()` operations.
- **R-CORE-009** SHOULD: Use atomic write patterns (e.g., `writeAgentsDirectoryAtomic`) for concurrent file operations during test execution or parallel agent configuration writes.

### Verify

```bash
# Check for synchronous fs methods in src/agents/ and src/core/
grep -r "readFileSync\|writeFileSync" src/agents/ src/core/ --include='*.ts' | wc -l | grep -q '^0$'

# Check for require('fs') without fs/promises
grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v "fs/promises" | wc -l | grep -q '^0$'

# Verify path module usage
grep -r "from ['\"]path['\"]" src/ tests/ --include='*.ts' | wc -l

# Check for third-party fs libraries
npm ls fs-extra graceful-fs 2>&1 | grep -q 'empty' || echo 'Third-party fs libs detected'
```

**Accept when:**
- All source files in src/agents/ and src/core/ use `fs/promises` for async file operations with zero synchronous fs method calls
- All path construction uses `path.join()` or `path.resolve()` with no string concatenation for file paths
- No third-party file system libraries (fs-extra, graceful-fs) appear in package.json dependencies for core functionality
- Test infrastructure consistently uses `os.tmpdir()` for temporary directories and `fs.rm()` with force option for cleanup
- All async file operations include proper error handling via try-catch blocks or .catch() handlers
- User-supplied paths are validated before use in path operations

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via grep patterns in CI pipeline checking for synchronous fs methods and string-based path concatenation is mandatory. Code review checklist must verify fs/promises and path module usage. Dependency audit in CI must reject PRs introducing third-party file system libraries without architectural review. CI build fails if synchronous fs methods detected in src/agents/ or src/core/ directories. PR review blocks merge if path operations use string concatenation instead of path module methods.
</enforcement>