# Standardize Node.js Core Module Usage for File System and Path Operations: Environment Variable Access

These rules are ALWAYS ACTIVE for all TypeScript modules in src/agents/, src/core/, src/cli/, src/paths/, src/vscode/ performing file system operations, configuration loading, path manipulation, or environment variable access.

### Rules

- **R-ENV-001** MUST: Environment variable access for configuration paths (e.g., XDG_CONFIG_HOME, HOME) MUST use `process.env` from Node.js runtime.
- **R-ENV-002** MUST: When accessing environment variables for configuration paths, provide fallback values using `os.homedir()` for cross-platform compatibility.
- **R-ENV-003** MUST: All file system operations MUST import from Node.js core modules ('fs', 'fs/promises', 'path', 'os') and MUST NOT import third-party file system libraries like fs-extra without documented exception approval.
- **R-ENV-004** MUST: Path operations MUST use `path.join()` for combining path segments and `path.resolve()` for absolute path resolution; MUST NOT concatenate paths with string operations or hardcoded separators.
- **R-ENV-005** MUST: JSON parsing of file contents MUST use native `JSON.parse()` after reading file content, with try-catch error handling including file path context.
- **R-ENV-006** SHOULD: For asynchronous operations, import from 'fs/promises' and use async/await syntax; for synchronous operations (e.g., CLI initialization), import from 'fs' and use *Sync methods explicitly.
- **R-ENV-007** SHOULD: Expand FileSystemUtils.ts to provide reusable utility functions for common patterns like recursive directory creation, safe file writes, and atomic operations to reduce boilerplate across modules.

### Verify

```bash
# Verify core module imports are present
grep -r "from ['\"]fs['\"]" src/ tests/ --include='*.ts' | wc -l

# Verify path module imports are present
grep -r "from ['\"]path['\"]" src/ tests/ --include='*.ts' | wc -l

# Verify no fs-extra imports exist
grep -r "from ['\"]fs-extra['\"]" src/ --include='*.ts' && echo 'FAIL: fs-extra found' || echo 'PASS: no fs-extra'

# Verify process.env usage for configuration paths
grep -r "process\\.env" src/ --include='*.ts' | grep -E '(XDG_CONFIG_HOME|HOME)' | wc -l

# Verify no hardcoded path separators in path construction
grep -r "['\"].*[/\\\\].*['\"]" src/ --include='*.ts' | grep -v 'path\\.' | wc -l
```

**Accept when:**
- All TypeScript modules performing file I/O import 'fs' or 'fs/promises' and 'path' from Node.js core modules, with no imports of third-party file system libraries like fs-extra
- Path operations use `path.join()` or `path.resolve()` methods rather than string concatenation, verified by absence of hardcoded path separators ('/' or '\\') in path construction
- Environment variable access for configuration uses `process.env` with appropriate fallbacks to `os.homedir()` for cross-platform compatibility
- JSON parsing of file contents uses native `JSON.parse()` after reading file content, with try-catch error handling including file path context
- All modules in scope (src/agents/, src/core/, src/cli/, src/paths/, src/vscode/) follow consistent patterns as demonstrated in FileSystemUtils.ts, ConfigLoader.ts, and UnifiedConfigLoader.ts

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Static analysis via grep commands, code review checklist verification, and integration test validation are mandatory before accepting changes to file system operations or environment variable access patterns.
</enforcement>