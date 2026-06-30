# Standardize Node.js Core Module Usage for File System and Environment Operations: Path Manipulation Operations

These rules are ALWAYS ACTIVE for all TypeScript modules in src/core/ and src/cli/ that perform file system operations, path manipulation, environment variable access, or configuration loading.

### Rules

- **R-PATH-001** MUST: Path manipulation operations MUST use the 'path' core module for all path resolution, joining, normalization, and parsing operations.
- **R-PATH-002** MUST: All file system operations in src/core/ and src/cli/ MUST use 'fs' or 'fs/promises' core modules with no third-party file system libraries.
- **R-PATH-003** MUST: All environment variable access MUST use process.env directly, checking process.env.XDG_CONFIG_HOME first for configuration directories before falling back to OS-specific defaults.
- **R-PATH-004** MUST: Security-sensitive operations such as symbolic link detection (isSymbolicLink, assertNotSymbolicLink) and directory traversal validation (assertContainingDirectoryInsideRoot) MUST use Node.js core module APIs.
- **R-PATH-005** SHOULD: Asynchronous file operations SHOULD use 'fs/promises' to maintain consistent async/await patterns; avoid callback-based 'fs' module APIs except where synchronous operations are explicitly required.
- **R-PATH-006** SHOULD: Core module operations SHOULD be wrapped in utility functions (e.g., findRulerDir, resolveProjectRootForRulerDir) that add project-specific validation and security checks rather than calling core modules directly throughout the codebase.
- **R-PATH-007** SHOULD: When traversing directory structures, SHOULD implement visited directory tracking (e.g., visitedDirectories.add(realDir)) to prevent infinite loops from circular symbolic links.

### Verify

```bash
# Verify core module imports are present and consistent
grep -r "from 'fs'" src/ && grep -r "from 'path'" src/ && grep -r "from 'os'" src/

# Verify XDG_CONFIG_HOME environment variable access pattern
grep -r "process\.env\.XDG_CONFIG_HOME" src/

# Verify security-sensitive functions are present
grep -r "isSymbolicLink\|assertNotSymbolicLink\|assertContainingDirectoryInsideRoot" src/

# Verify no third-party file system libraries are imported
grep -r "from 'fs-extra'\|from 'graceful-fs'" src/ && exit 1 || exit 0
```

**Accept when:**
- All file system operations in src/core/ and src/cli/ use 'fs' or 'fs/promises' core modules with no third-party file system libraries
- All path operations use the 'path' core module and all environment variable access uses process.env
- Configuration directory resolution checks process.env.XDG_CONFIG_HOME first
- Symbolic link validation functions (isSymbolicLink, assertNotSymbolicLink) and directory containment checks (assertContainingDirectoryInsideRoot) are present in the codebase
- No imports from 'fs-extra', 'graceful-fs', or other third-party file system abstraction libraries exist in src/

<enforcement>
Claude Code MUST NOT skip or defer verification. All file system and path operations MUST conform to these rules. Security-sensitive path validation logic MUST be reviewed before approval.
</enforcement>