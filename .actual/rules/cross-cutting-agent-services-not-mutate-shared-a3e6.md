# IAgent Service Interface Boundary and Directory-Scoped Resolution: Agent Services Not Mutate Shared Configuration

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-AGT-001** MUST_NOT: Agent services MUST NOT mutate shared configuration state across distinct directory boundaries.

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
Claude Code MUST NOT skip or defer verification. Verification is mandatory via automated CI test execution, peer code review, and static type verification.
</enforcement>