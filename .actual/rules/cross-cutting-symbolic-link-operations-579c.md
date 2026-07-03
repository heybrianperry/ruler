# Standardize Node.js Core Module Usage for File System and Environment Operations: Symbolic Link Operations

These rules are ALWAYS ACTIVE for all TypeScript modules in src/core/ and src/cli/ that perform file system operations, access environment variables, or handle path resolution and configuration loading.

### Rules

- **R-SYMLINK-001** MUST: Symbolic link operations MUST be explicitly handled with checks to prevent directory traversal vulnerabilities and ensure containment within expected root directories.
- **R-SYMLINK-002** MUST: All file system operations in src/core/ and src/cli/ MUST use 'fs' or 'fs/promises' core modules with no third-party file system libraries.
- **R-SYMLINK-003** MUST: All path operations MUST use the 'path' core module and all environment variable access MUST use process.env.
- **R-SYMLINK-004** MUST: Configuration directory resolution MUST check process.env.XDG_CONFIG_HOME first, then fall back to OS-specific defaults using the 'os' module.
- **R-SYMLINK-005** SHOULD: Use 'fs/promises' for asynchronous file operations to maintain consistent async/await patterns; avoid callback-based 'fs' module APIs except where synchronous operations are explicitly required.
- **R-SYMLINK-006** SHOULD: Wrap core module operations in utility functions (e.g., findRulerDir, resolveProjectRootForRulerDir) that add project-specific validation and security checks rather than calling core modules directly throughout the codebase.
- **R-SYMLINK-007** SHOULD: Implement visited directory tracking (e.g., visitedDirectories.add(realDir)) when traversing directory structures to prevent infinite loops from circular symbolic links.

### Verify

```bash
# Check for Node.js core module imports
grep -r "from 'fs'" src/ && grep -r "from 'path'" src/ && grep -r "from 'os'" src/

# Check for XDG_CONFIG_HOME environment variable usage
grep -r "process\.env\.XDG_CONFIG_HOME" src/

# Check for symbolic link validation functions
grep -r "isSymbolicLink\|assertNotSymbolicLink\|assertContainingDirectoryInsideRoot" src/
```

**Accept when:**
- All file system operations in src/core/ and src/cli/ use 'fs' or 'fs/promises' core modules with no third-party file system libraries
- All path operations use the 'path' core module and all environment variable access uses process.env
- Configuration directory resolution checks process.env.XDG_CONFIG_HOME
- Symbolic link validation functions are present in the codebase
- No imports from third-party file system libraries (fs-extra, graceful-fs, etc.) are found in scope

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis tools MUST scan import statements for 'fs', 'path', and 'os' core modules. Code review MUST verify Node.js core module usage for file system operations. CI pipeline MUST fail if third-party file system libraries are introduced without explicit exception approval. Security review is REQUIRED for any changes to symbolic link handling or directory traversal validation logic.
</enforcement>