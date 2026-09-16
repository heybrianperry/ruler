# Adoption of Core IAgent Interface for Agent Management: Discover Project Dependency Manifest Lock File

These rules are ALWAYS ACTIVE for modules within src/core and src/mcp that deal with agent definition, selection, configuration, or capabilities.

### Rules

- **R-IA-001** MUST: Discover the project's dependency manifest and lock file to determine the exact resolved version of any external dependencies before implementation.
- **R-IA-002** MUST: Find the dependency manifest in the repo.
- **R-IA-003** MUST: Identify the build tool from the manifest.
- **R-IA-004** MUST: Inspect the repository lock or resolution artifact to determine the exact resolved version.
- **R-IA-005** MUST: Look up the official documentation, changelog, or public API reference for that exact version.
- **R-IA-006** MUST: Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
- **R-IA-007** MUST: For version-sensitive behavior, re-run steps 3-5 (R-IA-004 to R-IA-006) per dependency at point of use.

### Verify

```bash
# Discover and run the project's static analysis tools.
# Discover and run the project's unit tests for agent-related modules.
# Discover and run the project's integration tests involving agent interactions.
```

**Accept when:**
- Static analysis reports no violations of IAgent interface usage.
- All agent-related unit and integration tests pass.
- New agent implementations correctly adhere to the IAgent contract.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>