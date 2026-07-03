# Isolate Environment Variable Access Behind Set-Based Deduplication in Filesystem Utilities: Modules Combining Security

These rules are ALWAYS ACTIVE for all filesystem utility modules that access environment variables for configuration path resolution, perform security-critical operations like symbolic link validation and directory traversal, or combine environment variable access with security operations.

### Rules

- **R-FSU-001** SHOULD: Modules combining security operations (symbolic link checks, root containment) with environment access SHOULD separate credential retrieval from filesystem operations through abstraction layers.

### Verify

```bash
# Verify no direct process.env access in FileSystemUtils.ts outside ConfigPathResolver
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
Claude Code MUST NOT skip or defer verification. Static analysis via ESLint rules prohibiting direct process.env access outside designated modules, code review checklists, and automated grep-based verification in CI pipeline are mandatory. Violations result in CI build failure and require security team review for exceptions.
</enforcement>