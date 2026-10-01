# Directory-Scoped Agent Resolution Caching with ConfigLoader and IAgent: Directory Scoped Service Definitions Not Retained

These rules are ALWAYS ACTIVE for operations that resolve or retrieve IAgent instances across directory-based configuration entries and service boundaries coordinating configuration loader results with agent execution contexts.

### Rules

- **R-DIR-001** SHOULD_NOT: Directory-scoped service definitions SHOULD NOT be retained across distinct command execution lifecycles without an explicit invalidation mechanism.
- **R-DIR-002** MANDATORY: Consumers must interact with agents exclusively through the IAgent interface abstraction.
- **R-DIR-003** MANDATORY: Before writing code that uses a versioned library, execute the lock-version grounding steps in order (find manifest, identify build tool, inspect lock/resolution artifact, look up official documentation for exact version, confirm every API/class/function exists, re-run per dependency at point of use).

### Verify

```bash
# Discover and execute the test suite declared in the project repository manifest to validate agent resolution caching.
# Run the repository static analysis and type verification scripts to ensure compliance with the IAgent interface contract.
```

**Accept when:**
- All tests verifying directory-keyed agent caching and retrieval pass successfully.
- Static type checking confirms all cached collections strictly satisfy the IAgent contract.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated test suites executed during continuous integration workflows and peer code review verifying directory-keyed agent lookup and avoidance of redundant resolution. Violation handling: Pull requests introducing redundant resolution or bypassing directory-keyed agent retrieval must be refactored before merging.
</enforcement>