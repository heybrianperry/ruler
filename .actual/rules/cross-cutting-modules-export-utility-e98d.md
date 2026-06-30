# Standardize Node.js Core Module Usage for File System and Environment Operations: Modules Export Utility

These rules are ALWAYS ACTIVE for all TypeScript modules in src/core/ and src/cli/ that perform file system operations, access environment variables, or resolve configuration paths.

### Rules

- **R-NODEJS-001** MUST: Use 'fs' or 'fs/promises' core modules for all file system operations; do not introduce third-party file system libraries such as fs-extra or graceful-fs.
- **R-NODEJS-002** MUST: Use the 'path' core module for all path manipulation and resolution operations.
- **R-NODEJS-003** MUST: Use the 'os' core module for operating system-specific operations such as home directory resolution.
- **R-NODEJS-004** MUST: Wrap core module operations in utility functions that add project-specific validation and security checks rather than calling core modules directly throughout the codebase.
- **R-NODEJS-005** MUST: Always use 'fs/promises' for asynchronous file operations to maintain consistent async/await patterns; avoid callback-based 'fs' module APIs except where synchronous operations are explicitly required.
- **R-NODEJS-006** MUST: When accessing environment variables for configuration directories, check process.env.XDG_CONFIG_HOME first, then fall back to OS-specific defaults using the 'os' module.
- **R-NODEJS-007** MUST: Implement visited directory tracking when traversing directory structures to prevent infinite loops from circular symbolic links.
- **R-NODEJS-008** MUST: Implement security-sensitive operations such as symbolic link detection (isSymbolicLink, assertNotSymbolicLink) and containment validation (assertContainingDirectoryInsideRoot) using Node.js core module APIs.
- **R-NODEJS-009** SHOULD: Use path.normalize and path.resolve consistently to handle platform-specific path differences across Windows, macOS, and Linux.

### Verify

```bash
# Verify core module imports are used
grep -r "from 'fs'" src/ && grep -r "from 'path'" src/ && grep -r "from 'os'" src/

# Verify XDG_CONFIG_HOME environment variable access pattern
grep -r "process\.env\.XDG_CONFIG_HOME" src/

# Verify security-sensitive functions are present
grep -r "isSymbolicLink\|assertNotSymbolicLink\|assertContainingDirectoryInsideRoot" src/

# Verify no third-party file system libraries are imported
grep -r "from 'fs-extra'\|from 'graceful-fs'" src/ && exit 1 || exit 0
```

**Accept when:**
- All file system operations in src/core/ and src/cli/ use 'fs' or 'fs/promises' core modules with no third-party file system libraries present.
- All path operations use the 'path' core module and all environment variable access uses process.env.
- Configuration directory resolution checks process.env.XDG_CONFIG_HOME and symbolic link validation functions (isSymbolicLink, assertNotSymbolicLink, assertContainingDirectoryInsideRoot) are present in the codebase.
- Asynchronous file operations consistently use 'fs/promises' with async/await patterns.
- Utility functions wrap core module operations with project-specific validation and security checks.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All file system and environment operations must conform to Node.js core module standards before code review approval.
</enforcement>