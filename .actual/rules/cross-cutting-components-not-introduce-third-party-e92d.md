# Adopt Node.js Core Filesystem Modules for File I/O Operations: Components Not Introduce Third Party Filesystem

These rules are ALWAYS ACTIVE for all agent implementations, MCP infrastructure, file-based configuration management components, and utility modules that perform filesystem operations in the Node.js runtime environment.

### Rules

- **R-FSCORE-001** MUST NOT: Components MUST NOT introduce third-party filesystem libraries for basic file I/O operations that can be accomplished with Node.js core modules (fs, fs/promises, path).
- **R-FSCORE-002** MUST: Agent implementations that read or write configuration files SHALL use Node.js core filesystem modules or the FileSystemUtils abstraction layer.
- **R-FSCORE-003** MUST: MCP propagation scripts that manage server definitions or configuration state SHALL use Node.js core filesystem modules or the FileSystemUtils abstraction layer.
- **R-FSCORE-004** MUST: Utility modules that provide filesystem abstractions or path operations SHALL import and use FileSystemUtils for common filesystem patterns.
- **R-FSCORE-005** MUST: Components that parse JSON data from files or persist JSON data to disk SHALL implement error handling for both filesystem errors and JSON parsing failures separately.
- **R-FSCORE-006** SHOULD: New filesystem operations SHALL be routed through the FileSystemUtils module to maintain consistency in error handling and API usage across subsystems.
- **R-FSCORE-007** SHOULD: Asynchronous filesystem operations SHALL use fs/promises with async/await patterns rather than callback-based fs methods.
- **R-FSCORE-008** MAY: Direct core module usage outside FileSystemUtils is permitted only with documented technical justification and senior engineer approval during code review.

### Verify

```bash
# Discover the project's static analysis configuration and execute the linting tool
# to verify that filesystem imports follow the established pattern of using core modules
# and FileSystemUtils.
find . -name '.eslintrc*' -o -name 'eslint.config.*' | head -1

# Locate the FileSystemUtils module in the core package
find . -path '*/core/*' -name '*FileSystemUtils*' -o -path '*/core/*' -name '*filesystem*' | grep -i util

# Search the codebase for direct imports of Node.js core filesystem modules
grep -r "from ['\"]fs['\"]\|from ['\"]fs/promises['\"]\|from ['\"]path['\"]\|require(['\"]fs['\"]\|require(['\"]fs/promises['\"]\|require(['\"]path['\"]" --include='*.ts' --include='*.js' | head -20

# Verify FileSystemUtils is imported alongside core modules
grep -r "FileSystemUtils" --include='*.ts' --include='*.js' | head -20

# Locate the project's test suite directory and run filesystem-related tests
find . -path '*/test*' -o -path '*/spec*' | grep -E '(test|spec)' | head -5

# Search for error handling patterns in filesystem operations
grep -r "catch.*Error\|try.*fs\|JSON.parse" --include='*.ts' --include='*.js' -A 2 | grep -E '(fs\.|FileSystemUtils|JSON.parse)' | head -20
```

**Accept when:**
- All agent implementations and MCP scripts import Node.js core filesystem modules (fs, fs/promises, path) for file I/O operations.
- FileSystemUtils module is imported and used for common filesystem patterns in components that perform file operations.
- Error handling is present for both filesystem errors and JSON parsing failures in all file read operations that parse JSON content.
- No third-party filesystem libraries (such as fs-extra, graceful-fs, or similar) are imported in agent implementations, MCP scripts, or configuration management components.
- Direct fs module usage without FileSystemUtils is accompanied by documented technical justification and evidence of code review approval.

<enforcement>
Clause MUST NOT skip or defer verification. All filesystem operations in scope MUST be reviewed against these rules before code merge. Violations block merge until corrected or approved exceptions are documented.
</enforcement>