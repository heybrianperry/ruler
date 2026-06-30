# Isolate Environment Variable Access Behind Set-Based Deduplication in Filesystem Utilities: Filesystem Utilities Cache

These rules are ALWAYS ACTIVE for all filesystem utility modules that access environment variables for configuration path resolution, perform security-critical filesystem operations (symbolic link validation, root containment checks), or generate error logging that may expose configuration paths.

### Rules

- **R-FSU-001** MAY: Filesystem utilities MAY cache resolved configuration paths to minimize repeated environment variable access.
- **R-FSU-002** MUST: All process.env access for configuration path resolution MUST route through dedicated accessor functions (e.g., ConfigPathResolver module) rather than direct environment variable access.
- **R-FSU-003** MUST: Error logging and diagnostic output MUST NOT expose raw environment variable values or unvalidated configuration paths in console.error calls.
- **R-FSU-004** SHOULD: Set-based deduplication patterns (visitedDirectories.add) SHOULD be maintained for directory traversal operations to prevent infinite loops in symbolic link resolution.
- **R-FSU-005** SHOULD: Path redaction utilities SHOULD mask sensitive portions of configuration paths in error messages while preserving debugging utility (e.g., show basename but mask parent directories).

### Verify

```bash
# Verify no direct process.env access outside ConfigPathResolver
grep -r 'process\.env\.' src/core/FileSystemUtils.ts | grep -v 'ConfigPathResolver' | wc -l | grep -q '^0$'

# Verify set-based deduplication via visitedDirectories.add is present
grep -r 'visitedDirectories\.add' src/core/FileSystemUtils.ts | wc -l | grep -q -v '^0$'

# Verify error logging does not expose raw environment variable values
grep -r 'console\.error.*\${.*env' src/core/ | wc -l | grep -q '^0$'
```

**Accept when:**
- All process.env access in FileSystemUtils.ts routes through dedicated accessor functions (zero direct access outside ConfigPathResolver)
- Set-based deduplication via visitedDirectories.add remains present and functional for directory traversal
- Error logging does not expose raw environment variable values in console.error calls
- ConfigPathResolver module exists with functions like getXdgConfigHome() that validate and sanitize environment variable values
- Unit tests verify environment variable access only occurs through designated accessor functions

<enforcement>
Claude Code MUST NOT skip or defer verification. All three bash verification commands MUST pass before accepting changes to filesystem utility modules. Static analysis via ESLint rules prohibiting direct process.env access outside designated modules is mandatory. Code review checklist verification of environment variable isolation is required. CI pipeline MUST fail if direct process.env access is detected outside ConfigPathResolver module.
</enforcement>