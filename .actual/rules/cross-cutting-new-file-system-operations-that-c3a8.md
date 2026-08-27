# Node.js Core File System Modules with Internal FileSystemUtils Abstraction: New File System Operations That Are

These rules are ALWAYS ACTIVE for all files in the `src/` directory that perform file I/O operations, including agent implementations, path utilities, MCP propagation logic, and configuration management modules.

### Rules

- **R-FS-001** SHOULD: New file system operations that are used across multiple components SHOULD be added to the FileSystemUtils module rather than duplicated in individual files.
- **R-FS-002** MUST: All file system operations MUST use Node.js core modules (path, fs, fs/promises) and MUST NOT introduce third-party file system libraries without architectural review and documented justification.
- **R-FS-003** SHOULD: When adding new methods to FileSystemUtils, they SHOULD be generic and reusable across multiple components; component-specific logic SHOULD remain in the component.
- **R-FS-004** SHOULD: New file system code SHOULD prefer async/await patterns with fs/promises to maintain consistency with modern JavaScript patterns.
- **R-FS-005** MUST: All text file read/write operations MUST explicitly specify file encoding (UTF-8 for JSON and configuration files) to avoid platform-dependent default encoding issues.
- **R-FS-006** MUST: Path operations MUST use the path module consistently to ensure cross-platform compatibility (Windows vs. Unix path separators).

### Verify

```bash
# Discover and inspect the project's static analysis configuration
find . -name '.eslintrc*' -o -name 'eslint.config.*' -o -name '.lintrc*' | head -5

# Search for all imports of file system modules
grep -r "require.*['\"]fs['\"]\|require.*['\"]fs/promises['\"]\|import.*from.*['\"]fs['\"]\|import.*from.*['\"]fs/promises['\"]" src/ --include='*.js' --include='*.ts' | wc -l

# Verify FileSystemUtils is imported alongside core modules
grep -r "FileSystemUtils" src/ --include='*.js' --include='*.ts' | head -10

# Check for third-party file system library imports (fs-extra, graceful-fs, etc.)
grep -r "require.*['\"]fs-extra['\"]\|require.*['\"]graceful-fs['\"]\|import.*from.*['\"]fs-extra['\"]\|import.*from.*['\"]graceful-fs['\"]" src/ --include='*.js' --include='*.ts'

# Locate and inspect the FileSystemUtils module
find . -path '*/src/core/FileSystemUtils*' -type f

# Run the project's test suite
npm test 2>&1 | grep -E '(PASS|FAIL|test|spec)' | head -20
```

**Accept when:**
- All file system operations in the codebase use Node.js core modules (path, fs, fs/promises) and no third-party file system libraries are present in dependencies
- The internal FileSystemUtils module is consistently imported and used across agent implementations and utilities for shared file system patterns
- All text file operations explicitly specify UTF-8 encoding
- Path operations consistently use the path module for cross-platform compatibility
- Static analysis or linting passes with no violations of file system module import patterns
- Automated tests verify that FileSystemUtils abstractions work correctly and that components use them consistently

<enforcement>
Claude Code MUST NOT skip or defer verification. All file system operations MUST conform to these rules before code is committed. Pull requests introducing third-party file system libraries or duplicating file system logic MUST be flagged and require architectural justification.
</enforcement>