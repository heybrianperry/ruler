# Adopt Node.js Core Filesystem Modules for File I/O Operations: Path Construction Manipulation Use Module Cross

These rules are ALWAYS ACTIVE for all agent implementations, MCP infrastructure, file-based configuration management components, and utility modules that perform filesystem operations in the Node.js runtime environment.

### Rules

- **R-FSCORE-001** SHOULD: Path construction and manipulation SHOULD use the path module's cross-platform methods rather than string concatenation to ensure compatibility across operating systems.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting tool
# to verify that filesystem imports follow the established pattern of using core modules
# and FileSystemUtils.
find . -name '.eslintrc*' -o -name 'tsconfig.json' | head -1

# Locate the project's test suite directory and run filesystem-related unit tests
# to confirm that FileSystemUtils abstractions work correctly and error handling
# patterns are consistent.
find . -type d -name '__tests__' -o -name 'test' -o -name 'tests' | head -1

# Search the codebase for direct imports of Node.js core filesystem modules
# and verify they are accompanied by proper error handling or routed through FileSystemUtils.
grep -r "require.*['\"]fs['\"]\|import.*from.*['\"]fs['\"]" --include='*.ts' --include='*.js' .

# Verify FileSystemUtils module exists and is properly exported
find . -path '*/core/*' -name '*FileSystemUtils*' -o -path '*/core/*' -name '*filesystem*'
```

**Accept when:**
- All agent implementations and MCP scripts import Node.js core filesystem modules (fs, fs/promises, path) for file I/O operations.
- FileSystemUtils module is imported and used for common filesystem patterns in components that perform file operations.
- Error handling is present for both filesystem errors and JSON parsing failures in all file read operations that parse JSON content.
- Path construction uses `path.join()`, `path.resolve()`, or `path.normalize()` instead of string concatenation.
- No direct string concatenation patterns like `dir + '/' + filename` are found in filesystem operation code.

<enforcement>
Claude Code MUST NOT skip or defer verification. All filesystem operations MUST be reviewed for compliance with cross-platform path construction requirements before code is merged.
</enforcement>