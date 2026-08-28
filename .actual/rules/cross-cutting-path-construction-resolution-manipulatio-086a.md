# Use Node.js Core Modules for File System Operations: Path Construction Resolution Manipulation Operations Use

These rules are ALWAYS ACTIVE for all files matching the configured scope: agent implementations that read or write configuration files, CLI command handlers that access the file system, core utilities that perform path resolution or file system queries, state management components that persist or retrieve data from disk, and any component that needs to verify file existence, read file metadata, or manipulate file system paths.

### Rules

- **R-CORE-FS-001** MUST: All path construction, resolution, and manipulation operations MUST use the Node.js core `path` module to ensure cross-platform compatibility.
- **R-CORE-FS-002** MUST: All file system read, write, and metadata operations MUST use Node.js core modules (`fs`, `fs/promises`) rather than third-party abstractions.
- **R-CORE-FS-003** SHOULD: Prefer promise-based async APIs (`fs/promises`) over callback-based APIs for better integration with async/await patterns.
- **R-CORE-FS-004** SHOULD: Separate file reading operations from parsing operations—read file content using core modules, then pass to format-specific parsers.
- **R-CORE-FS-005** MUST: Never use string concatenation for path construction; always use `path.join()` or `path.resolve()` functions.
- **R-CORE-FS-006** SHOULD: Avoid synchronous file system APIs in hot paths; restrict synchronous usage to initialization and CLI contexts only.
- **R-CORE-FS-007** MUST: Always validate and sanitize path inputs before file system operations to prevent path traversal attacks.
- **R-CORE-FS-008** SHOULD: Implement consistent error handling utilities that wrap common file operations and provide consistent error context.

### Verify

```bash
# Discover the project's static analysis configuration and execute the import analysis tool
# to verify that file system operations import from core modules
grep -r "from ['\"]fs['\"]" --include="*.ts" --include="*.js" .
grep -r "from ['\"]fs/promises['\"]" --include="*.ts" --include="*.js" .
grep -r "from ['\"]path['\"]" --include="*.ts" --include="*.js" .

# Locate the project's test suite and execute tests covering file system operations
npm test -- --testPathPattern="(fs|file|path)" 2>/dev/null || echo "Run project test suite to verify file system operations"

# Search the codebase for file system operation patterns
grep -r "require(['\"]fs['\"])" --include="*.ts" --include="*.js" .
grep -r "require(['\"]path['\"])" --include="*.ts" --include="*.js" .

# Verify no direct string concatenation for path construction
grep -r "\+ ['\"]" --include="*.ts" --include="*.js" . | grep -i path || echo "No obvious path string concatenation found"
```

**Accept when:**
- Static analysis confirms all file system read, write, and metadata operations use Node.js core modules (`fs`, `fs/promises`)
- Path manipulation operations consistently use the `path` module across all components
- No direct string concatenation is used for path construction in file system operations
- Promise-based async APIs are preferred over callback-based APIs in async contexts
- File reading and parsing operations are separated into distinct steps
- Synchronous file system APIs are restricted to initialization and CLI contexts
- Path inputs are validated before use in file system operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All file system operations must be audited against these rules before code is committed.
</enforcement>