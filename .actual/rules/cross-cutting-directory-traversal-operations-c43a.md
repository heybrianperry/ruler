# Isolate Environment Variable Access Behind Set-Based Deduplication in Filesystem Utilities: Directory Traversal Operations

These rules are ALWAYS ACTIVE for all filesystem utility modules that access environment variables for configuration path resolution, perform directory traversal operations, or combine security-critical filesystem operations with environment variable access.

### Rules

- **R-FSUTIL-001** MUST: Directory traversal operations that track visited paths MUST use Set-based deduplication (visitedDirectories.add) to prevent infinite loops and ensure deterministic path resolution.
- **R-FSUTIL-002** MUST: All process.env access for configuration path resolution MUST route through dedicated accessor functions (e.g., ConfigPathResolver module) rather than direct environment variable access.
- **R-FSUTIL-003** MUST: Error logging and diagnostic output MUST NOT expose raw environment variable values or configuration paths derived from process.env in console.error calls.
- **R-FSUTIL-004** SHOULD: Implement path redaction utilities in error messages that mask sensitive portions of configuration paths while preserving debugging utility (e.g., show basename but mask parent directories).

### Verify

```bash
# Verify no direct process.env access outside ConfigPathResolver
grep -r 'process\.env\.' src/core/FileSystemUtils.ts | grep -v 'ConfigPathResolver' | wc -l | grep -q '^0$'

# Verify set-based deduplication is present and functional
grep -r 'visitedDirectories\.add' src/core/FileSystemUtils.ts | wc -l | grep -q -v '^0$'

# Verify error logging does not expose raw environment values
grep -r 'console\.error.*\${.*env' src/core/ | wc -l | grep -q '^0$'
```

**Accept when:**
- All process.env access in FileSystemUtils.ts routes through dedicated accessor functions (zero direct access outside ConfigPathResolver)
- Set-based deduplication via visitedDirectories.add remains present and functional for directory traversal
- Error logging does not expose raw environment variable values in console.error calls
- ConfigPathResolver module exists with validated accessor functions for environment-derived paths

<enforcement>
Claude Code MUST NOT skip or defer verification. All three bash verification commands MUST pass before accepting changes to filesystem utilities that access environment variables or perform directory traversal operations.
</enforcement>