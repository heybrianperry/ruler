# Adopt Node.js Core Filesystem Modules for File I/O Operations: File Read Operations That Parse Json

These rules are ALWAYS ACTIVE for all agent implementations, MCP infrastructure, file-based configuration management components, and any code that reads or writes configuration files in JSON format within a Node.js runtime environment.

### Rules

- **R-FSIO-001** MUST: File read operations that parse JSON content MUST implement error handling for both file system errors and JSON parsing failures.

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
grep -r "require.*fs" --include="*.ts" --include="*.js" | grep -v FileSystemUtils
```

**Accept when:**
- All agent implementations and MCP scripts import Node.js core filesystem modules (fs, fs/promises, path) for file I/O operations.
- FileSystemUtils module is imported and used for common filesystem patterns in components that perform file operations.
- Error handling is present for both filesystem errors and JSON parsing failures in all file read operations that parse JSON content.
- Static analysis and linting rules detect direct filesystem module usage without proper error handling.
- Automated tests verify that filesystem operations follow established patterns and handle errors correctly.

<enforcement>
Clause MUST NOT skip or defer verification. Code review process checks for filesystem operation patterns and FileSystemUtils usage. Linting failures block merge until filesystem operations are corrected to follow the established pattern. Test failures for filesystem operations require fixes before code can be merged. Approved exceptions require documentation of technical justification in code comments and senior engineer approval during code review.
</enforcement>