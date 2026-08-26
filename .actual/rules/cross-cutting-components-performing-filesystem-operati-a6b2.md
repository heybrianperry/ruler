# Adopt Node.js Core Filesystem Modules for File I/O Operations: Components Performing Filesystem Operations Import Internal

These rules are ALWAYS ACTIVE for all agent implementations, MCP infrastructure, and file-based configuration management components performing filesystem operations in the Node.js runtime environment.

### Rules

- **R-FS-001** MUST: Components performing filesystem operations MUST import the internal FileSystemUtils module for common file operations to ensure consistent error handling and API patterns.
- **R-FS-002** MUST: Agent implementations that read or write configuration files MUST use FileSystemUtils abstractions rather than direct core module imports.
- **R-FS-003** MUST: MCP propagation scripts that manage server definitions or configuration state MUST import and use FileSystemUtils for filesystem operations.
- **R-FS-004** MUST: Error handling MUST be present for both filesystem errors and JSON parsing failures in all file read operations that parse JSON content.
- **R-FS-005** SHOULD: Use fs/promises for asynchronous operations and path for cross-platform path manipulation to maintain consistency with async/await patterns.
- **R-FS-006** SHOULD: Avoid mixing callback-based fs methods with promise-based code to maintain consistency across the codebase.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting tool
# to verify that filesystem imports follow the established pattern of using core modules and FileSystemUtils.
find . -name '.eslintrc*' -o -name 'eslint.config.*' | head -1

# Locate the project's test suite directory and run filesystem-related unit tests
# to confirm that FileSystemUtils abstractions work correctly and error handling patterns are consistent.
find . -type d -name '__tests__' -o -name 'test' -o -name 'tests' | head -1

# Search the codebase for direct imports of Node.js core filesystem modules
# and verify they are accompanied by proper error handling or routed through FileSystemUtils.
grep -r "from ['\"]fs['\"]\|from ['\"]fs/promises['\"]\|from ['\"]path['\"]" --include="*.ts" --include="*.js" .

# Locate the FileSystemUtils module in the core package
find . -path '*/core/*' -name '*FileSystemUtils*' -o -path '*/core/*' -name '*filesystem*'
```

**Accept when:**
- All agent implementations and MCP scripts import Node.js core filesystem modules (fs, fs/promises, path) for file I/O operations.
- FileSystemUtils module is imported and used for common filesystem patterns in components that perform file operations.
- Error handling is present for both filesystem errors and JSON parsing failures in all file read operations that parse JSON content.
- Static analysis and linting tools confirm that filesystem operations follow established patterns without direct core module usage outside the abstraction layer.
- Filesystem-related unit tests pass and verify that FileSystemUtils abstractions work correctly.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code that performs filesystem operations in the Node.js runtime environment.
</enforcement>