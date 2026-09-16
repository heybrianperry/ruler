# Adoption of Core Agent Modules and Input Validation: Agent Implementations Utilize Shared File System

These rules are ALWAYS ACTIVE for all new agent implementations and modifications to existing agent implementations.

### Rules

- **R-AGENT-001** SHOULD: Agent implementations SHOULD utilize shared file system utility modules for file operations.

### Verify

```bash
Discover and execute the project's static analysis tools.
Discover and execute the project's unit and integration tests for agent modules.
Discover and execute the project's dependency audit tools.
```

**Accept when:**
- Static analysis reports no violations of module import patterns for agent-related files.
- All agent-related unit and integration tests pass successfully.
- Dependency audit confirms consistent versioning of core modules and their dependencies.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>