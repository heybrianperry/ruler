# Adoption of FileSystemUtils Core Module: Agent Implementations Utilize Filesystemutils Abstractions Common

These rules are ALWAYS ACTIVE for agent adapter modules requiring filesystem interactions, workspace inspections, and configuration management components that read, parse, or persist local settings.

### Rules

- **R-FSU-001** MUST: Agent implementations MUST utilize FileSystemUtils abstractions for common file persistence, directory queries, and path manipulations rather than reimplementing ad-hoc filesystem routines.
- **R-FSU-002** MUST: When adding new filesystem capabilities needed across multiple agents, extend FileSystemUtils rather than embedding custom routines inside individual adapters.
- **R-FSU-003** MUST: Ensure all filesystem operations exported by FileSystemUtils handle asynchronous exceptions and normalize path separators consistently across operating environments.
- **R-FSU-004** MUST: Before writing code that uses a versioned library, execute the lock-version grounding process: find dependency manifest, identify build tool, inspect repository lock or resolution artifact to determine exact resolved version, look up official documentation for that exact version, and confirm every API/class/function exists in that version.

### Verify

```bash
# Discover and execute the project verification script from the repository manifest to validate module import compliance
# Execute the repository test runner discovered through workspace configuration to confirm filesystem utility integration tests pass
```

**Accept when:**
- All agent adapter modules and configuration handlers access shared filesystem operations exclusively through the FileSystemUtils core module.
- All automated integration and unit test suites defined in the repository pass without module resolution or filesystem access errors.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated static analysis checks verifying import boundaries across agent modules and peer code reviews confirming new agent adapters consume FileSystemUtils for common disk operations. Violations will block pull requests and must be refactored.
</enforcement>