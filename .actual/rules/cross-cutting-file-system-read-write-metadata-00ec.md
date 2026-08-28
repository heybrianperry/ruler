# Use Node.js Core Modules for File System Operations: File System Read Write Metadata Operations

These rules are ALWAYS ACTIVE for all files that perform file system read, write, and metadata operations, including agent implementations, CLI command handlers, core utilities, and state management components.

### Rules

- **R-FS-001** MUST: All file system read, write, and metadata operations MUST use the Node.js core file system module as the primary abstraction layer.
- **R-FS-002** MUST: When constructing file paths, always use the path module's `join()` or `resolve()` functions rather than string concatenation to ensure cross-platform compatibility.
- **R-FS-003** SHOULD: Prefer promise-based async APIs over callback-based APIs for better integration with async/await patterns.
- **R-FS-004** SHOULD: Separate file reading operations from parsing operations—read file content using core modules, then pass to format-specific parsers.
- **R-FS-005** MUST: Validate and sanitize path inputs before file system operations to prevent path traversal attacks.
- **R-FS-006** SHOULD: Avoid synchronous file system APIs in hot paths; restrict synchronous usage to initialization and CLI contexts.

### Verify

```bash
# Discover the project's static analysis configuration and execute the import analysis tool
# to verify that file system operations import from core modules
grep -r "require('fs'\|require(\"fs\|import.*from.*['\"]fs['\"]" --include="*.ts" --include="*.js" | head -20

# Search the codebase for file system operation patterns
# and verify they follow the path module usage
grep -r "path\.join\|path\.resolve" --include="*.ts" --include="*.js" | wc -l

# Verify no direct string concatenation is used for path construction
grep -r "\+ ['\"].*['\"]\|\+ \`" --include="*.ts" --include="*.js" | grep -i path | head -10

# Locate and execute the project's test suite covering file system operations
npm test -- --testPathPattern="(fs|file|path)" 2>/dev/null || echo "(test command varies by project)"

# Check for synchronous file system API usage outside initialization contexts
grep -r "readFileSync\|writeFileSync\|statSync" --include="*.ts" --include="*.js" | grep -v "// initialization\|// cli" | head -10
```

**Accept when:**
- Static analysis confirms all file system read, write, and metadata operations use Node.js core modules (`fs`, `path`)
- Path manipulation operations consistently use the `path` module across all components
- No direct string concatenation is used for path construction in file system operations
- File system operations are properly separated from parsing logic
- Synchronous file system APIs are restricted to initialization and CLI contexts
- Path inputs are validated before use in file system operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All file system operations must be audited against these rules before code is committed.
</enforcement>