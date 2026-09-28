# Adoption of FileSystemUtils Core Module: Deserialization Filesystem Artifacts Through Parsing Routines

These rules are ALWAYS ACTIVE for agent adapter modules requiring filesystem interactions and workspace inspections, and configuration management components that read, parse, or persist local settings.

### Rules

- **R-FS-001** SHOULD: Deserialization of filesystem artifacts through parsing routines SHOULD validate schema integrity and guard against malformed data structures prior to domain state mutation.
- **R-FS-002** MUST: When adding new filesystem capabilities needed across multiple agents, extend FileSystemUtils rather than embedding custom routines inside individual adapters.
- **R-FS-003** MUST: Ensure all filesystem operations exported by FileSystemUtils handle asynchronous exceptions and normalize path separators consistently across operating environments.

### Verify

```bash
# Discover and execute the project verification script from the repository manifest to validate module import compliance.
# Execute the repository test runner discovered through workspace configuration to confirm filesystem utility integration tests pass.
```

**Accept when:**
- All agent adapter modules and configuration handlers access shared filesystem operations exclusively through the FileSystemUtils core module.
- All automated integration and unit test suites defined in the repository pass without module resolution or filesystem access errors.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>