# Adopt Node.js Core Filesystem Modules for File I/O Operations: File Operations Agent Implementations Mcp Infrastructure

These rules are ALWAYS ACTIVE for all agent implementations, MCP infrastructure, file-based configuration management components, and utility modules that perform filesystem operations in the Node.js runtime environment.

### Rules

- **R-FSIO-001** MUST: All file I/O operations in agent implementations and MCP infrastructure MUST use Node.js core filesystem modules (fs, fs/promises, path) as the foundational API layer.
- **R-FSIO-002** MUST: FileSystemUtils module from the core package MUST be imported and used for common filesystem patterns in components that perform file operations (reading JSON files, writing configuration data, path resolution).
- **R-FSIO-003** MUST: Error handling MUST be present for both filesystem errors and JSON parsing failures in all file read operations that parse JSON content.
- **R-FSIO-004** SHOULD: Use fs/promises for asynchronous operations and path for cross-platform path manipulation to maintain consistency with async/await patterns.
- **R-FSIO-005** SHOULD: Avoid mixing callback-based fs methods with promise-based code to maintain consistency across the codebase.
- **R-FSIO-006** MAY: Direct core module usage outside FileSystemUtils is permitted only with documented technical justification and senior engineer approval during code review.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting tool
# to verify that filesystem imports follow the established pattern of using core modules and FileSystemUtils.
lint:filesystem

# Locate the project's test suite directory and run filesystem-related unit tests
# to confirm that FileSystemUtils abstractions work correctly and error handling patterns are consistent.
test:filesystem

# Search the codebase for direct imports of Node.js core filesystem modules
# and verify they are accompanied by proper error handling or routed through FileSystemUtils.
grep -r "require.*['\"]fs['\"]\|require.*['\"]fs/promises['\"]\|require.*['\"]path['\"]" --include="*.ts" --include="*.js" | grep -v node_modules
```

**Accept when:**
- All agent implementations and MCP scripts import Node.js core filesystem modules (fs, fs/promises, path) for file I/O operations.
- FileSystemUtils module is imported and used for common filesystem patterns in components that perform file operations.
- Error handling is present for both filesystem errors and JSON parsing failures in all file read operations that parse JSON content.
- Static analysis and linting rules confirm no direct filesystem module usage without proper error handling.
- Filesystem-related unit tests pass and verify that FileSystemUtils abstractions work correctly.

<enforcement>
Clause MUST NOT skip or defer verification. Code review process checks for filesystem operation patterns and FileSystemUtils usage. Static analysis and linting rules detect direct filesystem module usage without proper error handling. Automated tests verify that filesystem operations follow established patterns and handle errors correctly. Violations require refactoring or documented exceptions with senior engineer approval.
</enforcement>