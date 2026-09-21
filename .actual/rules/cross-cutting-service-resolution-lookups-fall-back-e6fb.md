# IAgent Service Interface Boundary and Directory-Scoped Resolution: Service Resolution Lookups Fall Back Root

These rules are ALWAYS ACTIVE for all files matching the invocation and execution of agent services across directory boundaries, configuration loading, and agent resolution workflows.

### Rules

- **R-SRV-001** SHOULD: Service resolution lookups fall back to root configuration directory entries when sub-directory specific agent configurations are not present.

### Verify

```bash
# Discover the project test runner from repository manifests and run all unit and integration test suites.
# Discover the project type checking and linting tools from repository configuration and verify type compliance for service boundary definitions.
```

**Accept when:**
- All test suites verifying multi-directory agent execution and reversion pass without failure.
- Static type analysis verifies that all agent instances implement the IAgent contract.
- Directory-scoped caching correctly separates agent configurations across distinct directory paths.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration, static type verification, and peer code review enforce these rules.
</enforcement>