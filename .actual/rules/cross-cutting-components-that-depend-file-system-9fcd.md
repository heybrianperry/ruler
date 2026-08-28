# Use Node.js Core Modules for File System Operations: Components That Depend File System Operations

These rules are ALWAYS ACTIVE for all components that depend on file system operations, including agent implementations, CLI infrastructure, core utilities, state management components, and any code that needs to verify file existence, read file metadata, or manipulate file system paths.

### Rules

- **R-FS-001** MUST: Components that depend on file system operations MUST handle errors from file system module calls and provide meaningful error context to callers.
- **R-FS-002** MUST: When constructing file paths, always use the path module's join or resolve functions rather than string concatenation to ensure cross-platform compatibility.
- **R-FS-003** SHOULD: Prefer promise-based async APIs over callback-based APIs for better integration with async/await patterns.
- **R-FS-004** SHOULD: Separate file reading operations from parsing operations—read file content using core modules, then pass to format-specific parsers.
- **R-FS-005** MUST: All file system read, write, and metadata operations MUST import from Node.js core modules (fs, path, etc.) and not from third-party file system abstraction libraries.

### Verify

```bash
# Discover the project's static analysis configuration and execute the import analysis tool
# to verify that file system operations import from core modules
grep -r "from ['\"]fs['\"]" --include="*.ts" --include="*.js" src/
grep -r "from ['\"]path['\"]" --include="*.ts" --include="*.js" src/

# Search the codebase for file system operation patterns and verify they follow
# the path module usage for path manipulation
grep -r "path\.join\|path\.resolve" --include="*.ts" --include="*.js" src/

# Verify no direct string concatenation is used for path construction
grep -r "\+ ['\"].*['\"]" --include="*.ts" --include="*.js" src/ | grep -i path || echo "No obvious path concatenation found"

# Locate and execute the project's test suite covering file system operations
npm test -- --testPathPattern=".*\.test\.ts$" 2>&1 | grep -E "PASS|FAIL"
```

**Accept when:**
- Static analysis confirms all file system read, write, and metadata operations use Node.js core modules (fs, path, etc.)
- Path manipulation operations consistently use the path module across all components
- No direct string concatenation is used for path construction in file system operations
- Error handling is present and provides meaningful context for file system operation failures
- Tests covering file system operations pass and verify core module API usage

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code that performs file system operations.
</enforcement>