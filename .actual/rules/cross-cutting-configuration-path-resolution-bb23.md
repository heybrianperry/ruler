# Isolate Environment Variable Access Behind Set-Based Deduplication in Filesystem Utilities: Configuration Path Resolution

These rules are ALWAYS ACTIVE for all filesystem utility modules that access environment variables for configuration path resolution, particularly those performing security-critical operations like symbolic link validation and directory traversal.

### Rules

- **R-CFG-001** SHOULD: Configuration path resolution SHOULD validate that environment-derived paths exist and are accessible before using them in security-critical operations.

### Verify

```bash
# Verify no direct process.env access outside ConfigPathResolver
grep -r 'process\.env\.' src/core/FileSystemUtils.ts | grep -v 'ConfigPathResolver' | wc -l | grep -q '^0$'

# Verify set-based deduplication via visitedDirectories.add remains present
grep -r 'visitedDirectories\.add' src/core/FileSystemUtils.ts | wc -l | grep -q -v '^0$'

# Verify error logging does not expose raw environment variable values
grep -r 'console\.error.*\${.*env' src/core/ | wc -l | grep -q '^0$'
```

**Accept when:**
- All process.env access in FileSystemUtils.ts routes through dedicated accessor functions (zero direct access outside ConfigPathResolver)
- Set-based deduplication via visitedDirectories.add remains present and functional for directory traversal
- Error logging does not expose raw environment variable values in console.error calls

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via ESLint rules prohibiting direct process.env access outside designated modules is mandatory. Code review checklists must verify environment variable isolation. CI pipeline must enforce grep-based verification for process.env patterns outside accessor functions, failing the build on violations.
</enforcement>