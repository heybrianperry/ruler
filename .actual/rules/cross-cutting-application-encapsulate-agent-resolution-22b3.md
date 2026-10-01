# Directory-Scoped Agent Resolution Caching with ConfigLoader and IAgent: Application Encapsulate Agent Resolution Boundaries Caching

These rules are ALWAYS ACTIVE for operations that resolve or retrieve IAgent instances across directory-based configuration entries and service boundaries coordinating configuration loader results with agent execution contexts.

### Rules

- **R-DIR-001** MUST: The application MUST encapsulate agent resolution boundaries by caching resolved IAgent collections in an in-memory lookup map keyed by configuration directory.
- **R-DIR-002** MUST: Directory-scoped agent caches MUST be scoped strictly to the lifecycle of the invoking command execution to prevent stale instance retention.
- **R-DIR-003** MUST: Consumers MUST interact with agents exclusively through the IAgent interface abstraction.

### Verify

```bash
# Discover and execute the test suite declared in the project repository manifest
# Run the repository static analysis and type verification scripts
```

**Accept when:**
- All tests verifying directory-keyed agent caching and retrieval pass successfully.
- Static type checking confirms all cached collections strictly satisfy the IAgent contract.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated test suites, peer code review, and static type checking verify compliance.
</enforcement>