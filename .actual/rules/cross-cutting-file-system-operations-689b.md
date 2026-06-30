# Standardize Node.js Core Module Usage for File System and Environment Operations: File System Operations

These rules are ALWAYS ACTIVE for all TypeScript modules in src/core/ and src/cli/ that perform file system operations, access environment variables, or manage configuration loading.

### Rules

- **R-FS-001** MUST: File system operations MUST use the 'fs' or 'fs/promises' core modules for all file and directory access, including reading, writing, and metadata operations.
- **R-FS-002** MUST: Always use 'fs/promises' for asynchronous file operations to maintain consistent async/await patterns; avoid callback-based 'fs' module APIs except where synchronous operations are explicitly required.
- **R-FS-003** MUST: Wrap core module operations in utility functions (e.g., findRulerDir, resolveProjectRootForRulerDir) that add project-specific validation and security checks rather than calling core modules directly throughout the codebase.
- **R-FS-004** MUST: When accessing environment variables, check process.env.XDG_CONFIG_HOME first for configuration directories, then fall back to OS-specific defaults using the 'os' module (e.g., os.homedir()).
- **R-FS-005** MUST: Implement visited directory tracking (e.g., visitedDirectories.add(realDir)) when traversing directory structures to prevent infinite loops from circular symbolic links.
- **R-FS-006** MUST: All path operations MUST use the 'path' core module and all environment variable access MUST use process.env.
- **R-FS-007** MUST: Security review is required for any changes to symbolic link handling or directory traversal validation logic.

### Verify

```bash
# Verify core module imports are present
grep -r "from 'fs'" src/ && grep -r "from 'path'" src/ && grep -r "from 'os'" src/

# Verify XDG_CONFIG_HOME environment variable access pattern
grep -r "process\.env\.XDG_CONFIG_HOME" src/

# Verify security-sensitive functions are present
grep -r "isSymbolicLink\|assertNotSymbolicLink\|assertContainingDirectoryInsideRoot" src/

# Verify no third-party file system libraries are imported
! grep -r "from 'fs-extra'\|from 'graceful-fs'" src/
```

**Accept when:**
- All file system operations in src/core/ and src/cli/ use 'fs' or 'fs/promises' core modules with no third-party file system libraries
- All path operations use the 'path' core module and all environment variable access uses process.env
- Configuration directory resolution checks process.env.XDG_CONFIG_HOME
- Symbolic link validation functions (isSymbolicLink, assertNotSymbolicLink, assertContainingDirectoryInsideRoot) are present in the codebase
- Asynchronous file operations use 'fs/promises' with async/await patterns
- Core module operations are wrapped in utility functions with project-specific validation
- Directory traversal includes visited directory tracking to prevent infinite loops

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All file system operations must be reviewed for compliance with R-FS-001 through R-FS-007 before code is accepted.
</enforcement>