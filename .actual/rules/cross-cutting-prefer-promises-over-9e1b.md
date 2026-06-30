# Standardize Node.js Core Module Usage for File System and Path Operations: Prefer Promises Over

These rules are ALWAYS ACTIVE for all agent implementations, core processors, utility modules, test infrastructure, and configuration loaders that perform file system and path operations in Node.js environments.

### Rules

- **R-NODEJS-001** MUST: Use 'fs/promises' for all new async file system operations instead of callback-based 'fs' module.
- **R-NODEJS-002** MUST: Never use synchronous fs methods (readFileSync, writeFileSync, etc.) in src/agents/ and src/core/ directories.
- **R-NODEJS-003** MUST: Construct all file paths using path.join() or path.resolve(); never concatenate paths with string literals using '/' or '\\'.
- **R-NODEJS-004** SHOULD: Import fs/promises using 'import fs from "fs/promises"' or named imports for tree-shaking benefits.
- **R-NODEJS-005** SHOULD: Wrap JSON.parse() in try-catch blocks when parsing configuration files read via fs/promises.
- **R-NODEJS-006** SHOULD: Use fs.access() with .then(() => true).catch(() => false) pattern for file existence checks.
- **R-NODEJS-007** MUST: Ensure test infrastructure cleanup uses fs.rm(path, { recursive: true, force: true }) in afterEach/afterAll hooks.
- **R-NODEJS-008** MUST: Validate user-supplied paths using isPathInsideOrEqual utility before path.join() or path.resolve() operations.
- **R-NODEJS-009** SHOULD: Use atomic write patterns (e.g., writeAgentsDirectoryAtomic) for concurrent file operations during agent configuration writes.
- **R-NODEJS-010** MUST: Not introduce third-party file system libraries (fs-extra, graceful-fs) for core functionality without architectural review.

### Verify

```bash
# Check for callback-based fs requires (should return 0)
grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v "fs/promises" | wc -l | grep -q '^0$'

# Check path module usage exists
grep -r "from ['\"]path['\"]" src/ tests/ --include='*.ts' | wc -l

# Check for synchronous fs methods in agent and core directories (should return 0)
grep -r "readFileSync\|writeFileSync" src/agents/ src/core/ --include='*.ts' | wc -l | grep -q '^0$'

# Check for third-party fs libraries in dependencies
npm ls fs-extra graceful-fs 2>&1 | grep -q 'empty' || echo 'Third-party fs libs detected'
```

**Accept when:**
- All source files in src/agents/ and src/core/ use 'fs/promises' for async file operations with zero synchronous fs method calls.
- All path construction uses path.join() or path.resolve() with no string concatenation for file paths.
- No third-party file system libraries (fs-extra, graceful-fs) appear in package.json dependencies for core functionality.
- Test infrastructure consistently uses os.tmpdir() for temporary directories and fs.rm() with force option for cleanup.
- All user-supplied paths are validated before use in path operations.
- JSON.parse() calls on file contents are wrapped in try-catch blocks.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement. Violations block PR merge and trigger automated remediation guidance.
</enforcement>