# AgentsMdAgent Module Adoption for Agent Implementations: Specialized Agent Implementations Not Bypass Shared

These rules are ALWAYS ACTIVE for all files matching the configured scope: development and maintenance of agent integration modules providing tool-specific or environment-specific agent workflows, and shared agent configuration, document generation, and interface definitions within the agent subsystem.

### Rules

- **R-AGT-001** SHOULD_NOT: Specialized agent implementations SHOULD NOT bypass the shared validation and parsing routines provided by the core agent abstractions.
- **R-AGT-002** MANDATORY: When constructing a new agent integration, extend the AgentsMdAgent base structure and fulfill the IAgent contract to inherit standard configuration lifecycle management.
- **R-AGT-003** MANDATORY: Delegate all markdown serialization and schema validation to the inherited AgentsMdAgent facilities rather than introducing local custom parsers.
- **R-AGT-004** MANDATORY: Prior to writing code that uses a versioned library, find the dependency manifest, identify the build tool, inspect the repository lock or resolution artifact to determine the exact resolved version, look up official documentation for that exact version, and confirm every API, class, or function exists in that exact version's documentation.

### Verify

```bash
# Discover the project test runner configuration from the repository manifest and execute the test suite covering agent integration modules.
# Discover the project static analysis and linting configuration from the repository manifest and execute validation across all agent integration components.
```

**Accept when:**
- All agent integration modules successfully instantiate and inherit from AgentsMdAgent without contract violations.
- All agent test suites pass cleanly with zero linting and type-checking diagnostics reported across agent implementations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated continuous integration pipelines validating static types, lint rules, and test suites across all agent modules, and by peer code review.
</enforcement>