# Standardize Node.js Core Module Usage for File System and Path Operations: Operating System Specific

These rules are ALWAYS ACTIVE for all TypeScript modules in src/agents/, src/core/, src/cli/, src/paths/, src/vscode/ performing file system operations, configuration loading, or path manipulation, including configuration loaders, file system utilities, agent implementations, CLI command handlers, and integration test harnesses.

### Rules

- **R-OS-001** SHOULD: Operating system-specific operations (home directory resolution, platform detection) SHOULD use the 'os' core module.

### Verify

```bash
# Verify fs core module usage
grep -r "from ['\"]fs['\"]" src/ tests/ --include='*.ts' | wc -l

# Verify path core module usage
grep -r "from ['\"]path['\"]" src/ tests/ --include='*.ts' | wc -l

# Verify no fs-extra usage
grep -r "from ['\"]fs-extra['\"]" src/ --include='*.ts' && echo 'FAIL: fs-extra found' || echo 'PASS: no fs-extra'

# Verify environment variable access for configuration
grep -r "process\\.env" src/ --include='*.ts' | grep -E '(XDG_CONFIG_HOME|HOME)' | wc -l
```

**Accept when:**
- All TypeScript modules performing file I/O import 'fs' or 'fs/promises' and 'path' from Node.js core modules, with no imports of third-party file system libraries like fs-extra
- Path operations use path.join() or path.resolve() methods rather than string concatenation, verified by absence of hardcoded path separators ('/' or '\\') in path construction
- Environment variable access for configuration uses process.env with appropriate fallbacks to os.homedir() for cross-platform compatibility
- JSON parsing of file contents uses native JSON.parse() after reading file content, with try-catch error handling including file path context
- Operating system-specific operations use the 'os' core module for home directory resolution and platform detection

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review acceptance.
</enforcement>