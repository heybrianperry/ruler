# AbstractAgent Core Module Adoption for Agent Implementations: Specialized Agent Implementations Extend Internal Abstractagent

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-AGT-001** MUST: All specialized agent implementations MUST extend the internal AbstractAgent base class to inherit foundational agent lifecycle and execution behavior.
- **R-AGT-002** MUST: Follow lock-version grounding before writing code that uses a versioned library (find manifest, identify build tool, inspect lock/resolution artifact, look up official documentation for exact version, confirm APIs exist, re-run per dependency at point of use).
- **R-AGT-003** MUST: Export newly implemented concrete agents through the primary module aggregation file alongside existing agent definitions.
- **R-AGT-004** MAY: Utilize a lightweight mock or test double strictly within automated test fixtures that cannot instantiate the full base class (Exception: EXC-20-001).

### Verify

```bash
# Discover and execute the repository's test runner to validate that all agent implementations successfully pass lifecycle and contract tests.
# Discover and execute the project's type-checking and linter suites to ensure all agent implementations strictly conform to AbstractAgent contracts.
```

**Accept when:**
- All concrete agent modules inherit from AbstractAgent and pass type verification without contract mismatches.
- All agent lifecycle and execution test suites pass without regressions across all agent variants.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>