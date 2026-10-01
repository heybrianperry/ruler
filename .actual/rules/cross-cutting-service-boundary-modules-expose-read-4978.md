# Directory-Scoped Agent Resolution Caching with ConfigLoader and IAgent: Service Boundary Modules Expose Read Only

These rules are ALWAYS ACTIVE for all files matching operations that resolve or retrieve IAgent instances across directory-based configuration entries and service boundaries coordinating configuration loader results with agent execution contexts.

### Rules

- **R-CACH-001** MAY: Service boundary modules MAY expose read-only access to cached agent instances to avoid redundant filesystem traversal and configuration parsing.
- **R-CACH-002** MANDATORY: Consumers must interact with agents exclusively through the IAgent interface abstraction.
- **R-CACH-003** MANDATORY: Directory-scoped agent caches must be scoped strictly to the lifecycle of the invoking command execution to prevent stale instance retention, clearing or re-instantiating directory mappings whenever configuration files are modified or reloaded.
- **R-CACH-004** MANDATORY (DISCOVERY POLICY): The consumer MUST discover tool names, file names, commands, package managers, and version numbers from the project repository.
- **R-CACH-005** MANDATORY (LOCK-VERSION GROUNDING): Before writing code that uses a versioned library, execute in order: (1) Find dependency manifest, (2) Identify build tool, (3) Inspect repository lock/resolution artifact, (4) Look up official docs for that exact version, (5) Confirm every API/class/function exists in that version, (6) Re-run steps 3-5 per dependency at point of use.

### Verify

```bash
# Discover and execute the test suite declared in the project repository manifest to validate agent resolution caching.
# Run the repository static analysis and type verification scripts to ensure compliance with the IAgent interface contract.
```

**Accept when:**
- All tests verifying directory-keyed agent caching and retrieval pass successfully.
- Static type checking confirms all cached collections strictly satisfy the IAgent contract.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated test suites and peer code reviews verify directory-keyed agent lookup and avoidance of redundant resolution. Pull requests introducing redundant resolution or bypassing directory-keyed agent retrieval must be refactored before merging.
</enforcement>