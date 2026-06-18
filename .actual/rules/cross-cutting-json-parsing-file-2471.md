# Standardize Node.js Core Module Usage for File System and Path Operations: Json Parsing File

These rules are ALWAYS ACTIVE for all TypeScript modules in src/agents/, src/core/, src/cli/, src/paths/, src/vscode/ performing file system operations, configuration loading, or path manipulation, including configuration loaders, file system utilities, agent implementations, CLI command handlers, and integration test harnesses.

### Rules

- **R-CORE-JSON-001** MUST: JSON parsing of file contents MUST use native JSON.parse() after reading file content as text.

### Verify

```bash
# Verify core module imports are present
grep -r "from ['\"]fs['\"]" src/ tests/ --include='*.ts' | wc -l
grep -r "from ['\"]path['\"]" src/ tests/ --include='*.ts' | wc -l

# Verify no third-party file system libraries
grep -r "from ['\"]fs-extra['\"]" src/ --include='*.ts' && echo 'FAIL: fs-extra found' || echo 'PASS: no fs-extra'

# Verify environment variable access patterns
grep -r "process\.env" src/ --include='*.ts' | grep -E '(XDG_CONFIG_HOME|HOME)' | wc -l

# Verify JSON.parse() usage with try-catch
grep -r "JSON\.parse" src/ --include='*.ts' -A 2 | grep -c "try\|catch"
```

**Accept when:**
- All TypeScript modules performing file I/O import 'fs' or 'fs/promises' and 'path' from Node.js core modules, with no imports of third-party file system libraries like fs-extra
- Path operations use path.join() or path.resolve() methods rather than string concatenation, verified by absence of hardcoded path separators ('/' or '\\') in path construction
- Environment variable access for configuration uses process.env with appropriate fallbacks to os.homedir() for cross-platform compatibility
- JSON parsing of file contents uses native JSON.parse() after reading file content, with try-catch error handling including file path context

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON.parse() usage patterns and try-catch error handling in all file I/O operations.
</enforcement>