# AgentsMdAgent Module Adoption for Agent Implementations: Engineers Discover Project Dependency Manifest Resolved

These rules are ALWAYS ACTIVE for development and maintenance of agent integration modules providing tool-specific or environment-specific agent workflows, and shared agent configuration, document generation, and interface definitions within the agent subsystem.

### Rules

- **R-AGT-001** MUST: Engineers MUST discover the project dependency manifest and resolved lock artifact to verify the exact resolved version of all supporting runtime dependencies before introducing or altering agent module dependencies.
- **R-AGT-002** MUST: When constructing a new agent integration, extend the AgentsMdAgent base structure and fulfill the IAgent contract to inherit standard configuration lifecycle management.
- **R-AGT-003** MUST: Delegate all markdown serialization and schema validation to the inherited AgentsMdAgent facilities rather than introducing local custom parsers.

### Verify

```bash
# Discover and execute test suite covering agent integration modules from the project repository manifest
# Discover and execute static analysis and linting across all agent integration components from the repository manifest
```

**Accept when:**
- All agent integration modules successfully instantiate and inherit from AgentsMdAgent without contract violations.
- All agent test suites pass cleanly with zero linting and type-checking diagnostics reported across agent implementations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated CI pipelines validating static types, lint rules, and test suites across all agent modules, as well as peer code review.
</enforcement>