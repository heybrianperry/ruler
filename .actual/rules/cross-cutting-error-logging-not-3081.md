# Isolate Environment Variable Access Behind Set-Based Deduplication in Filesystem Utilities: Error Logging Not

These rules are ALWAYS ACTIVE for all filesystem utility modules that access environment variables for configuration path resolution, perform security-critical filesystem operations (symbolic link validation, root containment checks), or generate error logging and diagnostic output that may expose configuration paths.

### Rules

- **R-FSUTIL-001** MUST_NOT: Error logging MUST NOT expose raw environment variable values or unvalidated configuration paths without redaction.
- **R-FSUTIL-002** MUST: All process.env access for configuration path resolution MUST route through dedicated accessor functions (e.g., ConfigPathResolver module).
- **R-FSUTIL-003** MUST: Direct process.env access outside designated accessor modules is prohibited in filesystem utilities.
- **R-FSUTIL-004** MUST: Set-based deduplication via visitedDirectories.add(realDir) pattern MUST remain present and functional for directory traversal operations.
- **R-FSUTIL-005** SHOULD: Error messages that reference configuration paths SHOULD use path redaction utilities that mask sensitive portions while preserving debugging utility (e.g., show basename but mask parent directories).
- **R-FSUTIL-006** SHOULD: Structured logging with separate channels for detailed diagnostics (restricted access) and sanitized user-facing errors SHOULD be implemented to balance security and troubleshooting needs.

### Verify

```bash
# Verify no direct process.env access outside ConfigPathResolver in FileSystemUtils
grep -r 'process\.env\.' src/core/FileSystemUtils.ts | grep -v 'ConfigPathResolver' | wc -l | grep -q '^0$'

# Verify set-based deduplication pattern is present
grep -r 'visitedDirectories\.add' src/core/FileSystemUtils.ts | wc -l | grep -q -v '^0$'

# Verify error logging does not expose raw environment variable values
grep -r 'console\.error.*\${.*env' src/core/ | wc -l | grep -q '^0$'
```

**Accept when:**
- All process.env access in FileSystemUtils.ts routes through dedicated accessor functions (zero direct access outside ConfigPathResolver)
- Set-based deduplication via visitedDirectories.add remains present and functional for directory traversal
- Error logging does not expose raw environment variable values in console.error calls
- No console.error statements interpolate environment-derived configuration paths without redaction

<enforcement>
Claude Code MUST NOT skip or defer verification. All three bash verification commands MUST pass before accepting changes to filesystem utilities that access environment variables. Violations detected in CI pipeline MUST cause build failure.
</enforcement>