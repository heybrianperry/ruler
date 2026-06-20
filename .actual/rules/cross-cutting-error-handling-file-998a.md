# Standardize Node.js Core Module Usage for File System and Path Operations: Error Handling File

These rules are ALWAYS ACTIVE for all TypeScript modules in src/agents/, src/core/, src/cli/, src/paths/, src/vscode/ performing file system operations, configuration loading, or path manipulation, including configuration loaders, file system utilities, agent implementations, CLI command handlers, and integration test harnesses.

### Rules

- **R-FS-001** SHOULD: Error handling for file operations SHOULD catch and log errors using console.warn or console.error with descriptive context.

### Verify

```bash
# Verify core module imports are used
grep -r "from ['\"]fs['\"]" src/ tests/ --include='*.ts' | wc -l
grep -r "from ['\"]path['\"]" src/ tests/ --include='*.ts' | wc -l

# Verify no third-party file system libraries
grep -r "from ['\"]fs-extra['\"]" src/ --include='*.ts' && echo 'FAIL: fs-extra found' || echo 'PASS: no fs-extra'

# Verify environment variable access patterns
grep -r "process\.env" src/ --include='*.ts' | grep -E '(XDG_CONFIG_HOME|HOME)' | wc -l

# Verify error handling in file operations
grep -r "try\|catch" src/ --include='*.ts' -A 2 | grep -E "(console\.(warn|error)|throw)" | wc -l
```

**Accept when:**
- All TypeScript modules performing file I/O import 'fs' or 'fs/promises' and 'path' from Node.js core modules, with no imports of third-party file system libraries like fs-extra
- Path operations use path.join() or path.resolve() methods rather than string concatenation, verified by absence of hardcoded path separators ('/' or '\\') in path construction
- Environment variable access for configuration uses process.env with appropriate fallbacks to os.homedir() for cross-platform compatibility
- JSON parsing of file contents uses native JSON.parse() after reading file content, with try-catch error handling including file path context
- File operation errors are caught and logged using console.warn or console.error with descriptive context including file path and operation type

<enforcement>
Claude Code MUST NOT skip or defer verification. All file operations must include error handling with descriptive logging as specified in R-FS-001.
</enforcement>