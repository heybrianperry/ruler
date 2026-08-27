# Node.js Core File System Modules with Internal FileSystemUtils Abstraction: Components Use Synchronous File System Sync

These rules are ALWAYS ACTIVE for all agent implementations, path utility modules, MCP propagation logic, and utilities in the src/ directory that perform file I/O operations.

### Rules

- **R-FS-001** MUST: Use Node.js core modules (path, fs, fs/promises) for all file system operations; do not introduce third-party file system libraries such as fs-extra or graceful-fs.
- **R-FS-002** MUST: Import and use the internal FileSystemUtils abstraction module (src/core/FileSystemUtils) for shared file system patterns including reading JSON files, ensuring directories exist, and resolving paths relative to project root.
- **R-FS-003** MAY: Use synchronous file system APIs (fs.*Sync methods) when operating in initialization or setup phases where blocking behavior is acceptable; document the rationale for synchronous usage.
- **R-FS-004** SHOULD: Prefer async/await patterns with fs/promises for new code to maintain consistency with modern JavaScript patterns; use synchronous APIs only when blocking behavior is explicitly required.
- **R-FS-005** MUST: Always specify file encoding explicitly when reading or writing text files (use UTF-8 for JSON and configuration files) to avoid platform-dependent default encoding issues.
- **R-FS-006** MUST: Ensure FileSystemUtils methods are generic and reusable across multiple components; component-specific logic must remain in the component, not in the shared utility module.
- **R-FS-007** SHOULD: Use the path module consistently for all path operations to ensure cross-platform compatibility (Windows vs. Unix path separators, case sensitivity).

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
find . -path "*/src/core/FileSystemUtils*" -type f

# Verify FileSystemUtils is consistently imported alongside core modules
grep -r "FileSystemUtils" src/ --include="*.ts" --include="*.js" | grep -v node_modules

# Locate and run the project's test suite
find . -name "package.json" -o -name "tsconfig.json" | xargs grep -l "test\|jest\|mocha" | head -1
```

**Accept when:**
- All file system operations in the codebase use Node.js core modules (path, fs, fs/promises) and no third-party file system libraries are present in dependencies
- The internal FileSystemUtils module is consistently imported and used across agent implementations and utilities for shared file system patterns
- All file encoding is explicitly specified (UTF-8) for text file operations
- Synchronous file system APIs are used only in initialization or setup phases with documented rationale
- Static analysis or linting passes with no violations of file system module import patterns
- FileSystemUtils abstractions are covered by unit tests and work correctly

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules R-FS-001 through R-FS-007 are mandatory for code review and CI pipeline enforcement.
</enforcement>