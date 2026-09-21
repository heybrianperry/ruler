# IAgent Service Interface Boundary and Directory-Scoped Resolution: Agent Service Implementations Implement Iagent Contract

These rules are ALWAYS ACTIVE for all code implementing, configuring, or invoking agent services across directory boundaries.

### Rules

- **R-AG-001** MUST: All agent service implementations MUST implement the IAgent contract to establish explicit service interface boundaries.

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
Claude Code MUST NOT skip or defer verification. All pull requests bypassing the service boundary or instantiating global state will be rejected.
</enforcement>