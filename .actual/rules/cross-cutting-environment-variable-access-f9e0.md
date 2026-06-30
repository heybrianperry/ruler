# Isolate Environment Variable Access Behind Set-Based Deduplication in Filesystem Utilities: Environment Variable Access

These rules are ALWAYS ACTIVE for all filesystem utility modules that access environment variables for configuration path resolution, particularly those performing security-critical operations like symbolic link validation and directory traversal.

### Rules

- **R-ENV-001** MUST: Environment variable access for configuration paths (e.g., `process.env.XDG_CONFIG_HOME`) MUST be isolated in dedicated accessor functions that validate and sanitize values before use.
- **R-ENV-002** MUST: All filesystem utility modules combining environment variable access with security-critical filesystem operations (symbolic link validation, root containment checks, directory traversal) MUST route environment access through designated accessor functions.
- **R-ENV-003** MUST: Error logging and diagnostic output MUST NOT expose raw environment variable values or configuration paths derived from environment variables in console.error calls.
- **R-ENV-004** SHOULD: Existing set-based deduplication patterns (e.g., `visitedDirectories.add(realDir)`) SHOULD be maintained and extended to credential source isolation boundaries.
- **R-ENV-005** SHOULD: Path redaction utilities SHOULD mask sensitive portions of configuration paths in error messages while preserving debugging utility (e.g., show basename but mask parent directories).

### Verify

```bash
# Verify no direct process.env access outside ConfigPathResolver in FileSystemUtils
grep -r 'process\.env\.' src/core/FileSystemUtils.ts | grep -v 'ConfigPathResolver' | wc -l | grep -q '^0$'

# Verify set-based deduplication via visitedDirectories.add remains present
grep -r 'visitedDirectories\.add' src/core/FileSystemUtils.ts | wc -l | grep -q -v '^0$'

# Verify error logging does not expose raw environment variable values
grep -r 'console\.error.*\${.*env' src/core/ | wc -l | grep -q '^0$'
```

**Accept when:**
- All `process.env` access in FileSystemUtils.ts routes through dedicated accessor functions (zero direct access outside ConfigPathResolver)
- Set-based deduplication via `visitedDirectories.add` remains present and functional for directory traversal
- Error logging does not expose raw environment variable values in `console.error` calls
- A dedicated ConfigPathResolver module exists with functions like `getXdgConfigHome()` that validate and sanitize environment variable values
- Unit tests verify environment variable access only occurs through designated accessor functions

<enforcement>
Claude Code MUST NOT skip or defer verification. All three bash verification commands MUST pass before accepting changes to filesystem utilities that access environment variables. Static analysis via ESLint rules prohibiting direct process.env access outside designated modules is mandatory. Code review checklist verification of environment variable isolation is required. CI pipeline must fail if direct process.env access is detected outside ConfigPathResolver module.
</enforcement>