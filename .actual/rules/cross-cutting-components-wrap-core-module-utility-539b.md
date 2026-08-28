# Use Node.js Core Modules for File System Operations: Components Wrap Core Module Utility Functions

These rules are ALWAYS ACTIVE for all files that perform file system operations, including agent implementations that read or write configuration files, CLI command handlers that access the file system, core utilities that perform path resolution or file system queries, state management components that persist or retrieve data from disk, and any component that needs to verify file existence, read file metadata, or manipulate file system paths.

### Rules

- **R-CORE-FS-001** MUST: All file system read, write, and metadata operations use Node.js core modules (`fs`, `fs/promises`, `path`) rather than external file system abstraction libraries.
- **R-CORE-FS-002** MUST: Path construction and manipulation use the `path` module's `join()` or `resolve()` functions rather than string concatenation to ensure cross-platform compatibility.
- **R-CORE-FS-003** SHOULD: Prefer promise-based async APIs (`fs/promises`) over callback-based APIs for better integration with async/await patterns.
- **R-CORE-FS-004** SHOULD: Separate file reading operations from parsing operations—read file content using core modules, then pass to format-specific parsers for configuration management.
- **R-CORE-FS-005** MAY: Components MAY wrap core module APIs in utility functions to provide domain-specific abstractions, but the underlying implementation MUST use core modules.
- **R-CORE-FS-006** MUST: Validate and sanitize path inputs before file system operations to prevent path traversal attacks; use path module functions to resolve and normalize paths.
- **R-CORE-FS-007** SHOULD: Implement consistent error handling across components that wrap common file operations and provide consistent error context.

### Verify

```bash
# Discover the project's static analysis configuration and execute the import analysis tool
# to verify that file system operations import from core modules
grep -r "from ['\"]fs" --include="*.ts" --include="*.js" | grep -v node_modules | head -20

# Verify path module usage for path manipulation
grep -r "from ['\"]path" --include="*.ts" --include="*.js" | grep -v node_modules | head -20

# Search for non-core file system imports that should be flagged
grep -r "from ['\"]fs-extra\|from ['\"]graceful-fs\|from ['\"]upath" --include="*.ts" --include="*.js" | grep -v node_modules

# Verify no string concatenation for path construction (basic check)
grep -r "\+ ['\"].*['\"].*path\|path.*\+ ['\"]" --include="*.ts" --include="*.js" | grep -v node_modules | head -10

# Locate and execute the project's test suite for file system operations
npm test -- --testPathPattern="(fs|file|path)" 2>/dev/null || echo "(no matching test suite found)"
```

**Accept when:**
- Static analysis confirms all file system read, write, and metadata operations use Node.js core modules (`fs`, `fs/promises`, `path`)
- Path manipulation operations consistently use the `path` module's `join()` or `resolve()` functions across all components
- No direct string concatenation is used for path construction in file system operations
- All file system operation imports originate from Node.js core modules, not external libraries
- Test suite for file system operations passes and verifies core module API usage

<enforcement>
Claude Code MUST NOT skip or defer verification. All file system operations MUST be audited against these rules before code is committed. Violations MUST be identified during code review and corrected before merge.
</enforcement>