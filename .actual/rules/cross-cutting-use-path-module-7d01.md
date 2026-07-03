# Standardize Node.js Core Module Usage for File System and Path Operations: Use Path Module

These rules are ALWAYS ACTIVE for all agent implementations, core processors, utility modules, test infrastructure, and configuration loaders that perform file system and path operations in Node.js environments.

### Rules

- **R-PATH-001** MUST: Use the 'path' module for all file path construction, resolution, and manipulation to ensure cross-platform compatibility.
- **R-PATH-002** MUST: Import fs/promises using 'import fs from "fs/promises"' or 'import { readFile, writeFile } from "fs/promises"' for tree-shaking benefits.
- **R-PATH-003** MUST: Always use path.join() or path.resolve() for constructing file paths; never concatenate strings with '/' or '\\'.
- **R-PATH-004** MUST: Use fs/promises for all async file operations; no synchronous fs methods (readFileSync, writeFileSync) in src/agents/ or src/core/ directories.
- **R-PATH-005** MUST: Wrap JSON.parse() in try-catch blocks to handle malformed configuration files gracefully after file reads.
- **R-PATH-006** MUST: Use fs.access() with .then(() => true).catch(() => false) pattern for existence checks rather than deprecated fs.exists().
- **R-PATH-007** MUST: Implement path validation using isPathInsideOrEqual utility or equivalent sanitization for all user-supplied paths before path.join() or path.resolve() operations.
- **R-PATH-008** SHOULD: Use atomic write patterns (e.g., writeAgentsDirectoryAtomic) for concurrent file operations during test execution or parallel agent configuration writes.
- **R-PATH-009** SHOULD: Ensure cleanup with fs.rm(path, { recursive: true, force: true }) in afterEach/afterAll hooks in test infrastructure to prevent test pollution.
- **R-PATH-010** SHOULD: Use os.tmpdir() for temporary directories in test infrastructure rather than hardcoded paths.

### Verify

```bash
# Check for synchronous fs methods in core directories
grep -r "readFileSync\|writeFileSync" src/agents/ src/core/ --include='*.ts' | wc -l | grep -q '^0$'

# Check for require('fs') without fs/promises
grep -r "require('fs')" src/ tests/ --include='*.ts' --include='*.js' | grep -v "fs/promises" | wc -l | grep -q '^0$'

# Verify path module imports exist
grep -r "from ['\"]path['\"]" src/ tests/ --include='*.ts' | wc -l

# Check for third-party fs libraries
npm ls fs-extra graceful-fs 2>&1 | grep -q 'empty' || echo 'Third-party fs libs detected'

# Check for string concatenation in path operations (basic heuristic)
grep -r "\+ ['\"].*[/\\\\]" src/agents/ src/core/ --include='*.ts' | grep -v "path\." | wc -l | grep -q '^0$'
```

**Accept when:**
- All source files in src/agents/ and src/core/ use 'fs/promises' for async file operations with zero synchronous fs method calls
- All path construction uses path.join() or path.resolve() with no string concatenation for file paths
- No third-party file system libraries (fs-extra, graceful-fs) appear in package.json dependencies for core functionality
- Test infrastructure consistently uses os.tmpdir() for temporary directories and fs.rm() with force option for cleanup
- All user-supplied paths are validated before use in path operations
- JSON.parse() calls are wrapped in try-catch blocks in configuration loaders

<enforcement>
Claude Code MUST NOT skip or defer verification. All file system operations in scope must comply with these rules before code is committed.
</enforcement>