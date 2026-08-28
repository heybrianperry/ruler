# Use Node.js Core Modules for File System Operations: Configuration File Parsing Separate Reading Core

These rules are ALWAYS ACTIVE for all files matching the configured scope: agent implementations that read or write configuration files, CLI command handlers that access the file system, core utilities that perform path resolution or file system queries, state management components that persist or retrieve data from disk, and any component that needs to verify file existence, read file metadata, or manipulate file system paths.

### Rules

- **R-CORE-FS-001** SHOULD: Configuration file parsing SHOULD separate file reading (using core modules) from format-specific parsing (using appropriate parser libraries) to maintain clear separation of concerns.

### Verify

```bash
# Discover the project's static analysis configuration and execute the import analysis tool to verify that file system operations import from core modules
# Locate the project's test suite and execute tests covering file system operations to verify they use core module APIs
# Search the codebase for file system operation patterns and verify they follow the path module usage for path manipulation and core file system module for I/O operations
```

**Accept when:**
- Static analysis confirms all file system read, write, and metadata operations use Node.js core modules
- Path manipulation operations consistently use the path module across all components
- No direct string concatenation is used for path construction in file system operations

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review process checks for proper use of core modules in file system operations. Static analysis tools scan for import patterns and flag non-core module usage for I/O. Automated tests verify file system operations use core module APIs. Violations identified in code review must be addressed before merge; static analysis failures block CI pipeline until resolved.
</enforcement>