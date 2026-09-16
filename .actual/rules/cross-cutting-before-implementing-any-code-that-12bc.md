# Adoption of Node.js Built-in Modules for System-Level Operations: Before Implementing Any Code That Uses

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-ADR-001** MUST: Before implementing any code that uses a versioned dependency, the exact resolved version MUST be identified from the project's lock file or resolution artifact.

### Verify

```bash
# Inspect relevant module import statements for `fs`, `path`, `child_process`, and `util`.
# Review code for direct usage of `child_process` APIs.
# Examine test setup files for system-level module imports.
```

**Accept when:**
- All file system operations use the `fs` module.
- All path manipulations use the `path` module.
- All child process executions use the `child_process` module.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>