# Node.js Core File System Modules with Internal FileSystemUtils Abstraction: Components Not Introduce Third Party File

These rules are ALWAYS ACTIVE for all agent implementations, path utility modules, MCP propagation logic, and utilities in the `src/` directory that perform file I/O operations.

### Rules

- **R-FSUTIL-001** MUST NOT: Components MUST NOT introduce third-party file system libraries as dependencies unless a specific capability gap in Node.js core modules is documented and approved.
- **R-FSUTIL-002** MUST: All file system operations MUST use Node.js core modules (`path`, `fs`, `fs/promises`) or the internal FileSystemUtils abstraction.
- **R-FSUTIL-003** MUST: Shared file system patterns MUST be implemented in the internal FileSystemUtils module to avoid code duplication across components.
- **R-FSUTIL-004** SHOULD: Prefer async/await patterns with `fs/promises` for new code to maintain consistency with modern JavaScript patterns.
- **R-FSUTIL-005** MUST: File encoding MUST be specified explicitly when reading or writing text files to avoid platform-dependent default encoding issues.
- **R-FSUTIL-006** SHOULD: Use the `path` module consistently for all path operations to ensure cross-platform compatibility.
- **R-FSUTIL-007** MUST: Component-specific file system logic MUST remain in the component; only generic and reusable patterns belong in FileSystemUtils.

### Verify

```bash
# Discover and inspect the project's static analysis configuration
find . -name '.eslintrc*' -o -name 'eslint.config.*' -o -name '.lintrc*' | head -5

# Search for all imports of file system modules
grep -r "require.*['\"]fs" src/ --include="*.js" --include="*.ts" | grep -v node_modules
grep -r "import.*from.*['\"]fs" src/ --include="*.js" --include="*.ts" | grep -v node_modules

# Verify no third-party file system libraries are imported
grep -r "require.*['\"]fs-extra\|graceful-fs\|upath" src/ --include="*.js" --include="*.ts" | grep -v node_modules
grep -r "import.*from.*['\"]fs-extra\|graceful-fs\|upath" src/ --include="*.js" --include="*.ts" | grep -v node_modules

# Locate the FileSystemUtils module
find . -path "*/core/FileSystemUtils*" -type f

# Verify FileSystemUtils is consistently imported alongside core modules
grep -r "FileSystemUtils" src/ --include="*.js" --include="*.ts" | grep -v node_modules

# Check for third-party file system libraries in dependency manifest
grep -E "fs-extra|graceful-fs|upath" package.json package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null || echo "No third-party fs libraries found"

# Run the project's test suite
npm test 2>/dev/null || yarn test 2>/dev/null || pnpm test 2>/dev/null || echo "Test command not found"
```

**Accept when:**
- All file system operations in the codebase use Node.js core modules (`path`, `fs`, `fs/promises`) and no third-party file system libraries are present in dependencies
- The internal FileSystemUtils module is consistently imported and used across agent implementations and utilities for shared file system patterns
- Static analysis or linting passes with no violations of file system module import patterns
- All file system operations specify encoding explicitly when reading or writing text files
- Async/await patterns with `fs/promises` are used for new code
- Component-specific logic is not duplicated in FileSystemUtils

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement.
</enforcement>