# Adoption of Core Agent Modules and Input Validation: Consumer Discover Ecosystem Lock File Resolve

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-AGENT-001** MUST: The consumer MUST discover the ecosystem's lock file and resolve the exact locked version of any versioned dependency before implementation.
- **R-AGENT-002** MUST: This ADR omits all tool names, file names, commands, package managers, and version numbers; the consumer MUST derive them from the project repository.
- **R-AGENT-003** MUST: Before writing code that uses a versioned library, the consumer MUST execute the LOCK-VERSION GROUNDING steps:
    1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
    2. Identify the build tool from the manifest.
    3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
    4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
    5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
    6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- **R-AGENT-004** MUST: New agents MUST start by importing the core agent interface and module to ensure adherence to the established structure.
- **R-AGENT-005** MUST: When handling configuration files or other structured inputs, the consumer MUST prioritize using existing parsing utilities (e.g., for JSON or TOML) to ensure consistent and secure data processing.

### Verify

```bash
# Discover and execute the project's static analysis tools.
# Discover and execute the project's unit and integration tests for agent modules.
# Discover and execute the project's dependency audit tools.
```

**Accept when:**
- Static analysis reports no violations of module import patterns for agent-related files.
- All agent-related unit and integration tests pass successfully.
- Dependency audit confirms consistent versioning of core modules and their dependencies.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>