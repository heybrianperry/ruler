# Adoption of Node.js Built-in Modules for System-Level Operations: Modules Requiring File System Access Utilize

These rules are ALWAYS ACTIVE for modules responsible for file system operations, process management, and core utilities.

### Rules

- **R-FS-001** MUST: Modules requiring file system access MUST utilize the Node.js `fs` module.

### Verify

```bash
# Inspect relevant module import statements for fs, path, child_process, and util.
# Review code for direct usage of child_process APIs.
# Examine test setup files for system-level module imports.
```

**Accept when:**
- All file system operations use the `fs` module.
- All path manipulations use the `path` module.
- All child process executions use the `child_process` module.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>