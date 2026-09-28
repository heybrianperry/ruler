# Adoption of FileSystemUtils Core Module: Modules Not Invoke Raw Low Level

These rules are ALWAYS ACTIVE for agent adapter modules requiring filesystem interactions, workspace inspections, and configuration management components that read, parse, or persist local settings.

### Rules

- **R-FSU-001** MUST_NOT: Modules MUST_NOT invoke raw low-level filesystem methods when an equivalent standardized helper is provided by FileSystemUtils.

### Verify

```bash
# Discover and execute the project verification script and repository test runner from the repository manifest to validate module import compliance and filesystem utility integration tests.
```

**Accept when:**
- All agent adapter modules and configuration handlers access shared filesystem operations exclusively through the FileSystemUtils core module.
- All automated integration and unit test suites defined in the repository pass without module resolution or filesystem access errors.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated static analysis checks and peer code reviews verify import boundaries, and PRs introducing duplicated filesystem helper functions or bypassing FileSystemUtils will be blocked.
</enforcement>