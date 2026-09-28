# Adoption of FileSystemUtils Core Module: Modules Requiring Filesystem Interactions Across Agent

These rules are ALWAYS ACTIVE for agent adapter modules requiring filesystem interactions and workspace inspections, and configuration management components that read, parse, or persist local settings.

### Rules

- **R-FS-001** MUST: Modules requiring filesystem interactions across agent adapters and configuration subsystems MUST import and coordinate file operations through the FileSystemUtils core module.
- **R-FS-002** MUST: When adding new filesystem capabilities needed across multiple agents, extend FileSystemUtils rather than embedding custom routines inside individual adapters.
- **R-FS-003** MUST: Ensure all filesystem operations exported by FileSystemUtils handle asynchronous exceptions and normalize path separators consistently across operating environments.
- **R-FS-004** MUST: Before writing code that uses a versioned library, execute in order: find dependency manifest, identify build tool, inspect repository lock/resolution artifact for exact version, look up official documentation for that exact version, confirm every API/class/function exists in that version, and re-run per dependency at point of use.

### Verify

```bash
# Discover and execute the project verification script from the repository manifest to validate module import compliance.
# Execute the repository test runner discovered through workspace configuration to confirm filesystem utility integration tests pass.
```

**Accept when:**
- All agent adapter modules and configuration handlers access shared filesystem operations exclusively through the FileSystemUtils core module.
- All automated integration and unit test suites defined in the repository pass without module resolution or filesystem access errors.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via automated static analysis checks, peer code reviews, and blocking pull requests that introduce duplicated filesystem helper functions or bypass FileSystemUtils.
</enforcement>