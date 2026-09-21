# IAgent Service Interface Boundary and Directory-Scoped Resolution: Consumer Modules Resolve Agent Service Instances

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-AGT-001** MUST: Consumer modules MUST resolve agent service instances per configuration directory scope using resolveSelectedAgents rather than instantiating service instances globally.

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