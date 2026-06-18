# Standardize Node.js Core Module Usage for File System and Path Operations: Modules Use Specialized

These rules are ALWAYS ACTIVE for all TypeScript modules in src/agents/, src/core/, src/cli/, src/paths/, src/vscode/ performing file system operations, configuration loading, or path manipulation.

### Rules

- **R-CORE-001** MUST: Use Node.js core modules ('fs', 'fs/promises', 'path', 'os') for all file I/O operations, path manipulation, and environment variable access.
- **R-CORE-002** MAY: Use specialized parsing libraries (@iarna/toml, zod) for non-JSON formats or validation, but file I/O MUST still use core modules.
- **R-CORE-003** MUST: Import from 'fs/promises' and use async/await syntax for asynchronous operations; import from 'fs' and use *Sync methods explicitly for synchronous operations only in initialization code.
- **R-CORE-004** MUST: Use path.join() for combining path segments and path.resolve() for absolute path resolution; never concatenate paths with string operations or hardcoded separators.
- **R-CORE-005** MUST: When accessing environment variables for configuration paths (XDG_CONFIG_HOME, HOME), provide fallback values using os.homedir() for cross-platform compatibility.
- **R-CORE-006** MUST: Wrap JSON.parse() calls in try-catch blocks with descriptive error messages including file path context.
- **R-CORE-007** MUST NOT: Import or use third-party file system libraries (fs-extra, graceful-fs) without documented exception approval.

### Verify

```bash
# Count fs core module imports
grep -r "from ['\"]fs['\"]" src/ tests/ --include='*.ts' | wc -l

# Count path core module imports
grep -r "from ['\"]path['\"]" src/ tests/ --include='*.ts' | wc -l

# Verify no fs-extra usage
grep -r "from ['\"]fs-extra['\"]" src/ --include='*.ts' && echo 'FAIL: fs-extra found' || echo 'PASS: no fs-extra'

# Count environment variable access with fallbacks
grep -r "process\.env" src/ --include='*.ts' | grep -E '(XDG_CONFIG_HOME|HOME)' | wc -l

# Verify path operations use path module methods
grep -r "path\.join\|path\.resolve" src/ --include='*.ts' | wc -l

# Check for hardcoded path separators in path construction
grep -r "['\"].*[/\\\\].*['\"]" src/ --include='*.ts' | grep -v "node_modules" | head -20
```

**Accept when:**
- All TypeScript modules performing file I/O import 'fs' or 'fs/promises' and 'path' from Node.js core modules, with no imports of third-party file system libraries like fs-extra
- Path operations use path.join() or path.resolve() methods rather than string concatenation, verified by absence of hardcoded path separators ('/' or '\\') in path construction
- Environment variable access for configuration uses process.env with appropriate fallbacks to os.homedir() for cross-platform compatibility
- JSON parsing of file contents uses native JSON.parse() after reading file content, with try-catch error handling including file path context
- Asynchronous file operations use fs/promises with async/await syntax
- Synchronous file operations are limited to initialization code and use *Sync methods explicitly

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-CORE rules are mandatory for modules in scope. Violations require architecture review and documented exception approval before merging.
</enforcement>