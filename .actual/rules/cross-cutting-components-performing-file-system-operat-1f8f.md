# Node.js Core File System Modules with Internal FileSystemUtils Abstraction: Components Performing File System Operations Use

These rules are ALWAYS ACTIVE for all components in the src/ directory that perform file system operations, including agent implementations, path utilities, MCP propagation logic, and configuration management modules.

### Rules

- **R-FSUTIL-001** MUST: All components performing file system operations MUST use Node.js core modules (path, fs, or fs/promises) as the primary file system API.
- **R-FSUTIL-002** MUST: File system operations in agent implementations (FirebenderAgent, ZedAgent, CrushAgent, GeminiCliAgent) MUST use the internal FileSystemUtils abstraction for shared patterns such as reading JSON files, ensuring directories exist, or resolving paths.
- **R-FSUTIL-003** MUST: No third-party file system libraries (such as fs-extra, graceful-fs, or similar) MUST be imported or used in components performing file system operations.
- **R-FSUTIL-004** SHOULD: New file system operations SHOULD use async/await patterns with fs/promises to maintain consistency with modern JavaScript patterns.
- **R-FSUTIL-005** MUST: File encoding MUST be specified explicitly when reading or writing text files; UTF-8 encoding MUST be used for JSON file operations.
- **R-FSUTIL-006** SHOULD: When adding new methods to FileSystemUtils, they SHOULD be generic and reusable across multiple components; component-specific logic SHOULD remain in the component.
- **R-FSUTIL-007** MUST: Path operations MUST use the path module consistently to ensure cross-platform compatibility (Windows vs. Unix path separators).

### Verify

```bash
# Discover and inspect the project's static analysis configuration
find . -name ".eslintrc*" -o -name "eslint.config.*" -o -name ".lintrc*" | head -5

# Search for all imports of file system modules
grep -r "require.*['\"]\(fs\|path\)['\"]" src/ --include="*.js" --include="*.ts" | head -20
grep -r "import.*from.*['\"]\(fs\|path\)['\"]" src/ --include="*.js" --include="*.ts" | head -20

# Verify no third-party file system libraries are imported
grep -r "require.*['\"]\(fs-extra\|graceful-fs\)['\"]" src/ --include="*.js" --include="*.ts"
grep -r "import.*from.*['\"]\(fs-extra\|graceful-fs\)['\"]" src/ --include="*.js" --include="*.ts"

# Locate FileSystemUtils module
find . -path "*/core/FileSystemUtils*" -type f

# Verify FileSystemUtils is imported alongside core modules
grep -r "FileSystemUtils" src/ --include="*.js" --include="*.ts" | head -10

# Check for synchronous file system operations and verify they are documented
grep -r "fs\.readFileSync\|fs\.writeFileSync\|fs\.readdirSync" src/ --include="*.js" --include="*.ts"

# Verify encoding is specified in file operations
grep -r "readFile\|writeFile" src/ --include="*.js" --include="*.ts" -A 2 | grep -E "encoding|utf" | head -10

# Run the project's test suite
npm test 2>&1 | tail -20
```

**Accept when:**
- All file system operations in the codebase use Node.js core modules (path, fs, fs/promises) and no third-party file system libraries are present in dependencies
- The internal FileSystemUtils module is consistently imported and used across agent implementations and utilities for shared file system patterns
- Static analysis or linting passes with no violations of file system module import patterns
- All file system operations that read or write text files specify UTF-8 encoding explicitly
- Path operations consistently use the path module for cross-platform compatibility
- New async file system operations use fs/promises with async/await patterns
- FileSystemUtils methods are generic and reusable; component-specific logic remains in components

<enforcement>
Claude Code MUST NOT skip or defer verification. All file system operations MUST conform to these rules before code is committed. Violations detected by static analysis tools MUST block the CI pipeline until resolved.
</enforcement>