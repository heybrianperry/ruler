# Standardize Node.js Core Module Usage for File System and Path Operations: Use Promises Asynchronous

These rules are ALWAYS ACTIVE for all agent implementations, core processors, utility modules, test infrastructure, and configuration loaders that perform file system or path operations in Node.js environments.

### Rules

- **R-PROMISES-001** MUST: Use `fs/promises` for all asynchronous file system operations requiring promise-based APIs.
- **R-PROMISES-002** MUST: Use `path.join()` or `path.resolve()` for constructing file paths; never concatenate strings with `/` or `\\`.
- **R-PROMISES-003** MUST: Wrap `JSON.parse()` in try-catch blocks when parsing configuration files read via `fs/promises`.
- **R-PROMISES-004** MUST: Ensure cleanup in test infrastructure using `fs.rm(path, { recursive: true, force: true })` in afterEach/afterAll hooks.
- **R-PROMISES-005** SHOULD: Use `fs.access()` with `.then(() => true).catch(() => false)` pattern for existence checks rather than deprecated `fs.exists()`.
- **R-PROMISES-006** SHOULD: Import fs/promises using `import fs from "fs/promises"` or named imports for tree-shaking benefits.
- **R-PROMISES-007** MUST: Implement proper error handling with try-catch blocks or `.catch()` handlers in all async file operations.
- **R-PROMISES-008** MUST: Validate user-supplied paths before `path.join()` or `path.resolve()` operations using path validation utilities.

### Verify

```bash
# Check for synchronous fs methods in agent and core directories
grep -r "readFileSync\|writeFileSync" src/agents/ src/core/ --include='*.ts' | wc -l | grep -q '^0$'

# Verify fs/promises imports are present
grep -r "from ['\"]fs/promises['\"]" src/ tests/ --include='*.ts' | wc -l

# Check for string-based path concatenation (basic heuristic)
grep -r "\+ ['\"][/\\\\]" src/agents/ src/core/ --include='*.ts' | wc -l | grep -q '^0$'

# Verify no third-party fs libraries in dependencies
npm ls fs-extra graceful-fs 2>&1 | grep -q 'empty' || echo 'Third-party fs libs detected'

# Check for require('fs') without /promises
grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v "fs/promises" | wc -l | grep -q '^0$'
```

**Accept when:**
- All source files in `src/agents/` and `src/core/` use `fs/promises` for async file operations with zero synchronous fs method calls
- All path construction uses `path.join()` or `path.resolve()` with no string concatenation for file paths
- No third-party file system libraries (fs-extra, graceful-fs) appear in package.json dependencies for core functionality
- Test infrastructure consistently uses `os.tmpdir()` for temporary directories and `fs.rm()` with force option for cleanup
- All async file operations include proper error handling via try-catch or .catch() handlers
- Configuration file parsing wraps `JSON.parse()` in try-catch blocks

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline validation. Violations block PR merge and trigger automated remediation guidance.
</enforcement>