# Standardize Node.js Core Module Usage for File System and Path Operations: Import Node Core

These rules are ALWAYS ACTIVE for all agent implementations, core processors, utility modules, test infrastructure, and configuration loaders that perform file system and path operations.

### Rules

- **R-NODE-CORE-001** MUST: Import Node.js core modules using bare specifiers without 'node:' prefix (e.g., 'fs/promises', 'path', 'os').
- **R-NODE-CORE-002** MUST: Use 'fs/promises' for all async file operations; no synchronous fs methods (readFileSync, writeFileSync) in src/agents/ or src/core/ directories.
- **R-NODE-CORE-003** MUST: Construct all file paths using path.join() or path.resolve(); never concatenate strings with '/' or '\\'.
- **R-NODE-CORE-004** MUST: Wrap JSON.parse() in try-catch blocks after file reads to handle malformed configuration files gracefully.
- **R-NODE-CORE-005** MUST: Use fs.access() with .then(() => true).catch(() => false) pattern for existence checks rather than deprecated fs.exists().
- **R-NODE-CORE-006** MUST: Implement path validation using isPathInsideOrEqual utility for all user-supplied paths before path.join() or path.resolve() operations.
- **R-NODE-CORE-007** MUST: Use atomic write patterns (e.g., writeAgentsDirectoryAtomic) for concurrent file operations; implement file locking or operation serialization where necessary.
- **R-NODE-CORE-008** MUST: Ensure test infrastructure cleanup with fs.rm(path, { recursive: true, force: true }) in afterEach/afterAll hooks to prevent test pollution.
- **R-NODE-CORE-009** SHOULD: Import fs/promises using named imports for tree-shaking benefits (e.g., 'import { readFile, writeFile } from "fs/promises"').
- **R-NODE-CORE-010** SHOULD: Enforce try-catch blocks or .catch() handlers in all async file operations to prevent unhandled promise rejections.

### Verify

```bash
# Check for synchronous fs methods in core directories
grep -r "readFileSync\|writeFileSync" src/agents/ src/core/ --include='*.ts' | wc -l | grep -q '^0$'

# Check for require('fs') without fs/promises
grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v "fs/promises" | wc -l | grep -q '^0$'

# Verify path module usage
grep -r "from ['\"]path['\"]" src/ tests/ --include='*.ts' | wc -l

# Check for third-party fs libraries
npm ls fs-extra graceful-fs 2>&1 | grep -q 'empty' || echo 'Third-party fs libs detected'

# Verify no string concatenation for paths (basic check)
grep -r "\+ ['\"][\\/]" src/agents/ src/core/ --include='*.ts' | wc -l | grep -q '^0$'
```

**Accept when:**
- All source files in src/agents/ and src/core/ use 'fs/promises' for async file operations with zero synchronous fs method calls
- All path construction uses path.join() or path.resolve() with no string concatenation for file paths
- No third-party file system libraries (fs-extra, graceful-fs) appear in package.json dependencies for core functionality
- Test infrastructure consistently uses os.tmpdir() for temporary directories and fs.rm() with force option for cleanup
- All async file operations include proper error handling via try-catch or .catch() handlers
- Path validation using isPathInsideOrEqual is applied to all user-supplied paths

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code in scope. Violations must be remediated before merge, with exceptions requiring documented architectural review approval and ADR-AUTO reference in code comments.
</enforcement>