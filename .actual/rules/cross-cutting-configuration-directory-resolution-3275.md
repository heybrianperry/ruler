# Standardize Node.js Core Module Usage for File System and Environment Operations: Configuration Directory Resolution

These rules are ALWAYS ACTIVE for all TypeScript modules in src/core/ that perform file system operations, all TypeScript modules in src/cli/ that access environment variables or file system, utility modules that provide path resolution or directory traversal, and functions that read or write markdown files, generated files, or configuration files.

### Rules

- **R-NODEJS-001** MUST: Configuration directory resolution MUST check process.env.XDG_CONFIG_HOME as the primary source for user configuration directory location.
- **R-NODEJS-002** MUST: All file system operations in src/core/ and src/cli/ MUST use 'fs' or 'fs/promises' core modules with no third-party file system libraries.
- **R-NODEJS-003** MUST: All path operations MUST use the 'path' core module and all environment variable access MUST use process.env.
- **R-NODEJS-004** MUST: Wrap core module operations in utility functions (e.g., findRulerDir, resolveProjectRootForRulerDir) that add project-specific validation and security checks rather than calling core modules directly throughout the codebase.
- **R-NODEJS-005** SHOULD: Always use 'fs/promises' for asynchronous file operations to maintain consistent async/await patterns; avoid callback-based 'fs' module APIs except where synchronous operations are explicitly required.
- **R-NODEJS-006** SHOULD: When accessing environment variables, check process.env.XDG_CONFIG_HOME first for configuration directories, then fall back to OS-specific defaults using the 'os' module (e.g., os.homedir()).
- **R-NODEJS-007** SHOULD: Implement visited directory tracking (e.g., visitedDirectories.add(realDir)) when traversing directory structures to prevent infinite loops from circular symbolic links.

### Verify

```bash
# Verify core module imports are used
grep -r "from 'fs'" src/ && grep -r "from 'path'" src/ && grep -r "from 'os'" src/

# Verify XDG_CONFIG_HOME is checked
grep -r "process\.env\.XDG_CONFIG_HOME" src/

# Verify security-sensitive functions are present
grep -r "isSymbolicLink\|assertNotSymbolicLink\|assertContainingDirectoryInsideRoot" src/
```

**Accept when:**
- All file system operations in src/core/ and src/cli/ use 'fs' or 'fs/promises' core modules with no third-party file system libraries
- All path operations use the 'path' core module and all environment variable access uses process.env
- Configuration directory resolution checks process.env.XDG_CONFIG_HOME and symbolic link validation functions are present in the codebase
- Utility functions wrap core module operations with project-specific validation and security checks
- Asynchronous file operations use 'fs/promises' with async/await patterns

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis tools MUST scan import statements for 'fs', 'path', and 'os' core modules. Code review MUST verify Node.js core module usage for file system operations. CI pipeline MUST fail if third-party file system libraries are introduced without explicit exception approval. Security review is REQUIRED for any changes to symbolic link handling or directory traversal validation logic.
</enforcement>