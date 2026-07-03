# Standardize Node.js Core Module Usage for File System and Path Operations: Use Synchronous Methods

These rules are ALWAYS ACTIVE for all agent implementations, core processors, utility modules, test infrastructure, and configuration loaders that perform file system and path operations.

### Rules

- **R-FSYNC-001** MUST_NOT: Use synchronous fs methods (e.g., fs.readFileSync, fs.writeFileSync) in agent implementations or core processors where async operations are available.
- **R-FSYNC-002** MUST: Import fs using 'import fs from "fs/promises"' or 'import { readFile, writeFile } from "fs/promises"' for tree-shaking benefits.
- **R-FSYNC-003** MUST: Use path.join() or path.resolve() for constructing file paths; never concatenate strings with '/' or '\\'.
- **R-FSYNC-004** MUST: Wrap JSON.parse() in try-catch blocks after file reads to handle malformed configuration files gracefully.
- **R-FSYNC-005** SHOULD: Use fs.access() with .then(() => true).catch(() => false) pattern for existence checks rather than deprecated fs.exists().
- **R-FSYNC-006** MUST: In test infrastructure, ensure cleanup with fs.rm(path, { recursive: true, force: true }) in afterEach/afterAll hooks to prevent test pollution.
- **R-FSYNC-007** MUST: Implement path validation using isPathInsideOrEqual utility or equivalent sanitization for all external path inputs to prevent path traversal vulnerabilities.
- **R-FSYNC-008** SHOULD: Use atomic write patterns (e.g., writeAgentsDirectoryAtomic) for concurrent file operations during test execution or parallel agent configuration writes.

### Verify

```bash
# Check for synchronous fs methods in agent and core directories
grep -r "readFileSync\|writeFileSync" src/agents/ src/core/ --include='*.ts' | wc -l | grep -q '^0$'

# Verify fs/promises imports are used
grep -r "from ['\"]fs/promises['\"]" src/ tests/ --include='*.ts' | wc -l

# Check for string-based path concatenation (basic heuristic)
grep -r "\+ ['\"][/\\\\]" src/agents/ src/core/ --include='*.ts' | wc -l | grep -q '^0$'

# Verify no third-party fs libraries in dependencies
npm ls fs-extra graceful-fs 2>&1 | grep -q 'empty' || echo 'Third-party fs libs detected'

# Verify path module usage
grep -r "from ['\"]path['\"]" src/ tests/ --include='*.ts' | wc -l
```

**Accept when:**
- All source files in src/agents/ and src/core/ use 'fs/promises' for async file operations with zero synchronous fs method calls (readFileSync, writeFileSync, etc.)
- All path construction uses path.join() or path.resolve() with no string concatenation for file paths
- No third-party file system libraries (fs-extra, graceful-fs) appear in package.json dependencies for core functionality
- Test infrastructure consistently uses os.tmpdir() for temporary directories and fs.rm() with force option for cleanup
- All async file operations include proper error handling via try-catch blocks or .catch() handlers
- External path inputs are validated before use in path.join() or path.resolve() operations

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via grep patterns in CI pipeline checking for synchronous fs methods and string-based path concatenation is mandatory. Code review checklist must verify fs/promises and path module usage. Dependency audit in CI must reject PRs introducing third-party file system libraries without architectural review. CI build fails if synchronous fs methods detected in src/agents/ or src/core/ directories. PR review blocks merge if path operations use string concatenation instead of path module methods.
</enforcement>