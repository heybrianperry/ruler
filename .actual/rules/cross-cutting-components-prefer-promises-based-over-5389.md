# Node.js Core File System Modules with Internal FileSystemUtils Abstraction: Components Prefer Promises Based Over Callback

These rules are ALWAYS ACTIVE for all agent implementations, path utility modules, MCP propagation logic, and utilities in the `src/` directory that perform file I/O operations.

### Rules

- **R-FSUTIL-001** MUST: Use Node.js core modules (`path`, `fs`, `fs/promises`) for all file system operations; do not introduce third-party file system libraries such as `fs-extra` or `graceful-fs`.
- **R-FSUTIL-002** SHOULD: Prefer the promises-based API (`fs/promises`) over callback-based APIs for new asynchronous file system operations to maintain consistency with modern async/await patterns.
- **R-FSUTIL-003** SHOULD: Use the internal `FileSystemUtils` abstraction module for shared file system patterns such as reading JSON files, ensuring directories exist, or resolving paths relative to project root.
- **R-FSUTIL-004** MUST: Always specify file encoding explicitly when reading or writing text files; use UTF-8 encoding for JSON operations to avoid platform-dependent default encoding issues.
- **R-FSUTIL-005** SHOULD: Use the `path` module consistently for all path operations to ensure cross-platform compatibility between Windows and Unix-like systems.
- **R-FSUTIL-006** MAY: Use synchronous file system APIs only when blocking behavior is explicitly required; document the rationale for synchronous operations in code comments.
- **R-FSUTIL-007** SHOULD: Keep FileSystemUtils methods generic and reusable across multiple components; component-specific file system logic should remain in the component, not in the shared utility module.

### Verify

```bash
# Discover and inspect the project's static analysis configuration
find . -name ".eslintrc*" -o -name "eslint.config.*" -o -name "tsconfig.json" | head -5

# Search for all imports of file system modules
grep -r "from ['\"]fs" src/ --include="*.ts" --include="*.js" | grep -v node_modules
grep -r "require(['\"]fs" src/ --include="*.ts" --include="*.js" | grep -v node_modules

# Verify no third-party file system libraries are imported
grep -r "from ['\"]fs-extra" src/ --include="*.ts" --include="*.js"
grep -r "from ['\"]graceful-fs" src/ --include="*.ts" --include="*.js"
grep -r "require(['\"]fs-extra" src/ --include="*.ts" --include="*.js"
grep -r "require(['\"]graceful-fs" src/ --include="*.ts" --include="*.js"

# Locate and inspect the FileSystemUtils module
find . -path "*/core/FileSystemUtils*" -type f

# Verify FileSystemUtils is consistently imported alongside core modules
grep -r "FileSystemUtils" src/ --include="*.ts" --include="*.js" | grep -v node_modules

# Check for callback-based fs API usage in new code
grep -r "fs\.readFile[^S]" src/ --include="*.ts" --include="*.js"
grep -r "fs\.writeFile[^S]" src/ --include="*.ts" --include="*.js"

# Verify encoding is specified in file operations
grep -r "readFile\|writeFile\|readFileSync\|writeFileSync" src/ --include="*.ts" --include="*.js" -A 1 | grep -i "encoding"

# Run the project's test suite if available
if [ -f "package.json" ]; then npm test 2>/dev/null || yarn test 2>/dev/null || pnpm test 2>/dev/null; fi
```

**Accept when:**
- All file system operations in the codebase use Node.js core modules (`path`, `fs`, `fs/promises`) and no third-party file system libraries are present in dependencies
- The internal `FileSystemUtils` module is consistently imported and used across agent implementations and utilities for shared file system patterns
- All asynchronous file system operations use `fs/promises` with async/await syntax rather than callback-based APIs
- File encoding is explicitly specified (typically UTF-8) in all text file read/write operations
- The `path` module is used for all path operations to ensure cross-platform compatibility
- Static analysis or linting passes with no violations of file system module import patterns
- Automated tests verify that `FileSystemUtils` abstractions work correctly and that components use them consistently

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement.
</enforcement>