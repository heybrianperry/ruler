# Standardize Node.js Core Module Usage for File System and Environment Operations: Operating System Environment

These rules are ALWAYS ACTIVE for all TypeScript modules in src/core/ that perform file system operations, all TypeScript modules in src/cli/ that access environment variables or file system, utility modules that provide path resolution or configuration loading, and functions that read or write markdown files or generated files.

### Rules

- **R-OS-001** MUST: Operating system environment access MUST use the 'os' core module for OS-specific information and the 'process.env' object for environment variable access.
- **R-OS-002** MUST: All file system operations in src/core/ and src/cli/ MUST use 'fs' or 'fs/promises' core modules with no third-party file system libraries.
- **R-OS-003** MUST: All path operations MUST use the 'path' core module.
- **R-OS-004** MUST: Configuration directory resolution MUST check process.env.XDG_CONFIG_HOME first, then fall back to OS-specific defaults using the 'os' module (e.g., os.homedir()).
- **R-OS-005** MUST: Wrap core module operations in utility functions (e.g., findRulerDir, resolveProjectRootForRulerDir) that add project-specific validation and security checks rather than calling core modules directly throughout the codebase.
- **R-OS-006** SHOULD: Use 'fs/promises' for asynchronous file operations to maintain consistent async/await patterns; avoid callback-based 'fs' module APIs except where synchronous operations are explicitly required.
- **R-OS-007** SHOULD: Implement visited directory tracking (e.g., visitedDirectories.add(realDir)) when traversing directory structures to prevent infinite loops from circular symbolic links.

### Verify

```bash
# Verify core module imports are used
grep -r "from 'fs'" src/ && grep -r "from 'path'" src/ && grep -r "from 'os'" src/

# Verify XDG_CONFIG_HOME environment variable access pattern
grep -r "process\.env\.XDG_CONFIG_HOME" src/

# Verify security-sensitive functions are present
grep -r "isSymbolicLink\|assertNotSymbolicLink\|assertContainingDirectoryInsideRoot" src/
```

**Accept when:**
- All file system operations in src/core/ and src/cli/ use 'fs' or 'fs/promises' core modules with no third-party file system libraries
- All path operations use the 'path' core module and all environment variable access uses process.env
- Configuration directory resolution checks process.env.XDG_CONFIG_HOME and symbolic link validation functions are present in the codebase
- No imports from third-party file system libraries (fs-extra, graceful-fs, etc.) are found in scope
- Symbolic link validation and directory traversal checks are implemented using core module APIs

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis tools MUST scan import statements for 'fs', 'path', and 'os' core modules. Code review MUST verify Node.js core module usage for file system operations. CI pipeline MUST fail if third-party file system libraries are introduced without explicit exception approval.
</enforcement>