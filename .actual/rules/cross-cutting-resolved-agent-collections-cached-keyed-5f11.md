# IAgent Service Interface Boundary and Directory-Scoped Resolution: Resolved Agent Collections Cached Keyed Directory

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-AG-001** MUST: Resolved agent collections MUST be cached and keyed by directory path within selectedAgentsByRulerDir to prevent redundant resolution across identical configuration boundaries.

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
Claude Code MUST NOT skip or defer verification.
</enforcement>