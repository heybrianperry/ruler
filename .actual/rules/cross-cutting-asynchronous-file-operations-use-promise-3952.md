# Adopt Node.js Core Filesystem Modules for File I/O Operations: Asynchronous File Operations Use Promise Based

These rules are ALWAYS ACTIVE for all agent implementations, MCP infrastructure, file-based configuration management components, and any code that performs filesystem operations in the Node.js runtime environment.

### Rules

- **R-FSIO-001** SHOULD: Asynchronous file operations SHOULD use the promise-based API from `fs/promises` rather than callback-based `fs` methods to align with modern async/await patterns.
- **R-FSIO-002** MUST: Use Node.js core filesystem modules (`fs`, `fs/promises`, `path`) for all file I/O operations without introducing external dependencies for basic file operations.
- **R-FSIO-003** MUST: Import and use the FileSystemUtils module from the core package for common filesystem patterns including reading JSON files, writing configuration data, and path resolution.
- **R-FSIO-004** MUST: Implement error handling for both filesystem errors and JSON parsing failures separately in all file read operations that parse JSON content.
- **R-FSIO-005** SHOULD: Avoid mixing callback-based `fs` methods with promise-based code to maintain consistency across the codebase.
- **R-FSIO-006** MAY: Document technical justification in code comments when bypassing FileSystemUtils or using alternative filesystem libraries, with approval from a senior engineer or architect.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting tool
# to verify that filesystem imports follow the established pattern of using core modules
# and FileSystemUtils.
lint:filesystem

# Locate the project's test suite directory and run filesystem-related unit tests
# to confirm that FileSystemUtils abstractions work correctly and error handling
# patterns are consistent.
test:filesystem

# Search the codebase for direct imports of Node.js core filesystem modules
# and verify they are accompanied by proper error handling or routed through FileSystemUtils.
grep -r "require.*fs['\"]\|from.*['\"]fs['\"]" --include="*.ts" --include="*.js" | grep -v node_modules
```

**Accept when:**
- All agent implementations and MCP scripts import Node.js core filesystem modules (`fs`, `fs/promises`, `path`) for file I/O operations.
- FileSystemUtils module is imported and used for common filesystem patterns in components that perform file operations.
- Error handling is present for both filesystem errors and JSON parsing failures in all file read operations that parse JSON content.
- No callback-based `fs` methods are mixed with promise-based code in the same operation.
- All filesystem operations follow established patterns verified by code review and linting.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code that performs filesystem operations in the Node.js runtime. Violations must be addressed before code can be merged.
</enforcement>