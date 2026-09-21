# IAgent Service Interface Boundary and Directory-Scoped Resolution: Before Introducing Updating External Dependencies That

These rules are ALWAYS ACTIVE for all code involving agent services, configuration loading, multi-directory agent execution/reversion workflows, and external dependency management.

### Rules

- **R-ADR-001** MUST: Before introducing or updating external dependencies that support agent services, the consumer MUST inspect the repository dependency manifest and lock artifact to verify and adhere to the exact resolved versions.
- **R-ADR-002** MUST: Execute lock-version grounding steps in order (find manifest, identify build tool, inspect lock artifact for exact version, check public docs/API reference for that exact version, confirm APIs exist, re-run per dependency at point of use).
- **R-ADR-003** MUST: Implement service resolution using a mapping structure that associates directory paths with their respective resolved IAgent collection.
- **R-ADR-004** MUST: Ensure lookup functions check the specific directory entry first and query root configuration entries as a fallback when the key is absent.
- **R-ADR-005** MUST: Ensure all agent instances implement the IAgent contract.

### Verify

```bash
# Discover the project build tool and lock file, and run all unit and integration test suites
# Discover the project type checking and linting tools from repository configuration and verify type compliance
```

**Accept when:**
- All test suites verifying multi-directory agent execution and reversion pass without failure.
- Static type analysis verifies that all agent instances implement the IAgent contract.
- Directory-scoped caching correctly separates agent configurations across distinct directory paths.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated CI test execution, peer code review, and static type verification enforce adherence to service interfaces and dependency rules.
</enforcement>