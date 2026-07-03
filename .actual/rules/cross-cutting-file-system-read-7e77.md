# Standardize Node.js Core Module Usage for File System and Path Operations: File System Read

These rules are ALWAYS ACTIVE for all TypeScript modules in src/agents/, src/core/, src/cli/, src/paths/, src/vscode/ performing file system operations, configuration loading, or path manipulation, including configuration loaders, file system utilities, agent implementations, CLI command handlers, and integration test harnesses.

### Rules

- **R-FS-001** MUST: All file system read and write operations MUST use Node.js core modules 'fs' or 'fs/promises' rather than third-party file system abstraction libraries.
- **R-FS-002** MUST: Path operations MUST use path.join() or path.resolve() methods rather than string concatenation; never hardcode path separators ('/' or '\\').
- **R-FS-003** MUST: Environment variable access for configuration paths (XDG_CONFIG_HOME, HOME) MUST provide fallback values using os.homedir() for cross-platform compatibility.
- **R-FS-004** MUST: JSON.parse() calls on file contents MUST be wrapped in try-catch blocks with descriptive error messages including file path context.
- **R-FS-005** SHOULD: Prefer fs/promises with async/await syntax for all I/O operations except initialization code; use fs *Sync methods only when synchronous operations are explicitly required.
- **R-FS-006** MAY: Create internal utility functions in FileSystemUtils.ts for common patterns (recursive directory creation, safe file writes, atomic operations) to reduce boilerplate and ensure consistency.

### Verify

```bash
# Verify core module imports are present
grep -r "from ['\"]fs['\"]" src/ tests/ --include='*.ts' | wc -l

# Verify path module imports are present
grep -r "from ['\"]path['\"]" src/ tests/ --include='*.ts' | wc -l

# Verify no third-party file system libraries are used
grep -r "from ['\"]fs-extra['\"]" src/ --include='*.ts' && echo 'FAIL: fs-extra found' || echo 'PASS: no fs-extra'

# Verify environment variable access for configuration
grep -r "process\.env" src/ --include='*.ts' | grep -E '(XDG_CONFIG_HOME|HOME)' | wc -l

# Verify no hardcoded path separators in path construction
grep -r "['\"].*[/\\\\].*['\"]" src/ --include='*.ts' | grep -v "http" | grep -v "comment" || echo 'PASS: no obvious hardcoded separators'
```

**Accept when:**
- All TypeScript modules performing file I/O import 'fs' or 'fs/promises' and 'path' from Node.js core modules, with no imports of third-party file system libraries like fs-extra
- Path operations use path.join() or path.resolve() methods rather than string concatenation, verified by absence of hardcoded path separators in path construction
- Environment variable access for configuration uses process.env with appropriate fallbacks to os.homedir() for cross-platform compatibility
- JSON parsing of file contents uses native JSON.parse() after reading file content, with try-catch error handling including file path context
- Asynchronous file operations use fs/promises with async/await syntax; synchronous operations are explicitly documented and limited to initialization code

<enforcement>
Claude Code MUST NOT skip or defer verification. All file system operations MUST comply with R-FS-001 through R-FS-005. Violations discovered during code review or static analysis MUST be addressed before merge.
</enforcement>