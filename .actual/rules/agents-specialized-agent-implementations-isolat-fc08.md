# AgentsMdAgent Module Adoption for Agent Implementations: Specialized Agent Implementations Isolate Target Specific

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-AGT-001** SHOULD: Specialized agent implementations SHOULD isolate target-specific protocol serialization from core agent behavior by delegating document normalization directly to AgentsMdAgent.

### Verify

```bash
# Discover the project test runner configuration from the repository manifest and execute the test suite covering agent integration modules.
# Discover the project static analysis and linting configuration from the repository manifest and execute validation across all agent integration components.
```

**Accept when:**
- All agent integration modules successfully instantiate and inherit from AgentsMdAgent without contract violations.
- All agent test suites pass cleanly with zero linting and type-checking diagnostics reported across agent implementations.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration pipelines validate static types, lint rules, and test suites across all agent modules. Pull requests introducing divergent agent configuration parsers or bypassing AgentsMdAgent are blocked from merge until refactored to use the shared module.
</enforcement>