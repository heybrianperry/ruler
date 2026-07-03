# Standardize Node.js Core Module Usage for File System and Path Operations: Path Construction Resolution

These rules are ALWAYS ACTIVE for all TypeScript modules in src/agents/, src/core/, src/cli/, src/paths/, src/vscode/ performing file system operations, configuration loading, or path manipulation, including configuration loaders, file system utilities, agent implementations, CLI command handlers, and integration test harnesses.

### Rules

- **R-PATH-001** MUST: All path construction, resolution, and manipulation operations MUST use the 'path' core module to ensure cross-platform compatibility.
- **R-PATH-002** MUST: Never concatenate paths with string operations or hardcoded separators ('/' or '\\'); always use path.join() for combining path segments and path.resolve() for absolute path resolution.
- **R-PATH-003** MUST: All file system operations MUST import from 'fs' or 'fs/promises' (Node.js core modules) and MUST NOT import third-party file system libraries like fs-extra without documented exception approval.
- **R-PATH-004** MUST: When accessing environment variables for configuration paths (XDG_CONFIG_HOME, HOME), provide fallback values using os.homedir() for cross-platform compatibility.
- **R-PATH-005** MUST: JSON.parse() calls on file contents MUST be wrapped in try-catch blocks with descriptive error messages including file path context.
- **R-PATH-006** SHOULD: Prefer fs/promises for all asynchronous I/O operations using async/await syntax; reserve synchronous fs methods for initialization code only.
- **R-PATH-007** SHOULD: Expand FileSystemUtils.ts to provide reusable utility functions for common patterns (recursive directory creation, safe file writes, atomic operations) to reduce boilerplate and ensure consistency.

### Verify

```bash
# Count fs core module imports
grep -r "from ['\"]fs['\"]" src/ tests/ --include='*.ts' | wc -l

# Count path core module imports
grep -r "from ['\"]path['\"]" src/ tests/ --include='*.ts' | wc -l

# Verify no fs-extra imports
grep -r "from ['\"]fs-extra['\"]" src/ --include='*.ts' && echo 'FAIL: fs-extra found' || echo 'PASS: no fs-extra'

# Count environment variable usage for config paths
grep -r "process\.env" src/ --include='*.ts' | grep -E '(XDG_CONFIG_HOME|HOME)' | wc -l

# Check for hardcoded path separators in path construction
grep -r "['\"].*[/\\\\].*['\"]" src/ --include='*.ts' | grep -v "http" | grep -v "url" | wc -l
```

**Accept when:**
- All TypeScript modules performing file I/O import 'fs' or 'fs/promises' and 'path' from Node.js core modules, with no imports of third-party file system libraries like fs-extra
- Path operations use path.join() or path.resolve() methods rather than string concatenation, verified by absence of hardcoded path separators ('/' or '\\') in path construction
- Environment variable access for configuration uses process.env with appropriate fallbacks to os.homedir() for cross-platform compatibility
- JSON parsing of file contents uses native JSON.parse() after reading file content, with try-catch error handling including file path context
- FileSystemUtils.ts serves as the canonical reference implementation for core module usage patterns including error handling, directory traversal, and backup operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All path construction and file system operations MUST comply with R-PATH-001 through R-PATH-007. Violations discovered during code review MUST result in rejection or refactoring task creation. Exceptions require architecture review and ADR amendment.
</enforcement>