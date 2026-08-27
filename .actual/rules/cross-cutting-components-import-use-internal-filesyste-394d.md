# Node.js Core File System Modules with Internal FileSystemUtils Abstraction: Components Import Use Internal Filesystemutils Module

These rules are ALWAYS ACTIVE for all agent implementations, path utility modules, MCP propagation logic, and utilities in the `src/` directory that perform file I/O operations.

### Rules

- **R-FSUTIL-001** MUST: Components MUST import and use the internal FileSystemUtils module for shared file system operations that have been abstracted into that module.
- **R-FSUTIL-002** MUST: All file system operations in agent implementations (FirebenderAgent, ZedAgent, CrushAgent, GeminiCliAgent) MUST use Node.js core modules (path, fs, fs/promises) either directly or through FileSystemUtils abstractions.
- **R-FSUTIL-003** MUST: No third-party file system libraries (such as fs-extra, graceful-fs) MUST be imported or used in components; all file system operations MUST rely on Node.js core modules or FileSystemUtils.
- **R-FSUTIL-004** MUST: File encoding MUST be specified explicitly when reading or writing text files to avoid platform-dependent default encoding issues; UTF-8 encoding MUST be used for JSON file operations.
- **R-FSUTIL-005** SHOULD: New file system operations SHOULD use async/await patterns with fs/promises for consistency with modern JavaScript patterns; synchronous APIs SHOULD only be used when blocking behavior is explicitly required and documented.
- **R-FSUTIL-006** SHOULD: New methods added to FileSystemUtils SHOULD be generic and reusable across multiple components; component-specific logic SHOULD remain in the component, not in the shared utility module.

### Verify

```bash
# Discover and inspect the project's static analysis configuration
find . -name '.eslintrc*' -o -name 'eslint.config.*' -o -name '.lintrc*' | head -5

# Search for all imports of file system modules
grep -r "require.*['\"]fs['\"]\|require.*['\"]fs/promises['\"]\|require.*['\"]path['\"]\|import.*from.*['\"]fs['\"]\|import.*from.*['\"]fs/promises['\"]\|import.*from.*['\"]path['\"]" src/ --include="*.js" --include="*.ts" | wc -l

# Verify FileSystemUtils is imported alongside core modules
grep -r "FileSystemUtils" src/ --include="*.js" --include="*.ts" | head -10

# Check for third-party file system library imports
grep -r "require.*['\"]fs-extra['\"]\|require.*['\"]graceful-fs['\"]\|import.*from.*['\"]fs-extra['\"]\|import.*from.*['\"]graceful-fs['\"]" src/ --include="*.js" --include="*.ts"

# Locate and inspect the FileSystemUtils module
find . -path "*/core/FileSystemUtils*" -type f

# Run the project's test suite
npm test 2>/dev/null || yarn test 2>/dev/null || pnpm test 2>/dev/null
```

**Accept when:**
- All file system operations in the codebase use Node.js core modules (path, fs, fs/promises) and no third-party file system libraries are present in dependencies
- The internal FileSystemUtils module is consistently imported and used across agent implementations and utilities for shared file system patterns
- Static analysis or linting passes with no violations of file system module import patterns
- All file encoding specifications are explicit (UTF-8 for JSON operations)
- Async/await patterns with fs/promises are used for new file system operations
- FileSystemUtils methods are generic and reusable, not component-specific

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory and MUST be verified before accepting changes to file system operations in the codebase.
</enforcement>