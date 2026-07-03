# Standardize Agent Configuration Loading with Input Validation and Contract Exports: Agent Implementations Export

These rules are ALWAYS ACTIVE for all agent implementations and configuration loaders within the system, specifically targeting files in `src/agents/` directory and `src/core/UnifiedConfigLoader.ts`.

### Rules

- **R-AGENT-001** MUST: All agent implementations MUST export a public contract interface that implements IAgent.
- **R-AGENT-002** MUST: All JSON.parse() and parseTOML() calls in agent implementations and configuration loaders MUST be wrapped in try-catch blocks with agent-specific error context.
- **R-AGENT-003** MUST: Configuration validation MUST occur immediately after parsing and before passing to applyRulerConfig or other business logic.
- **R-AGENT-004** MUST: Agent classes MUST be exported as named exports matching the class name (e.g., `export class OpenCodeAgent implements IAgent`).
- **R-AGENT-005** SHOULD: Configuration schema expectations SHOULD be documented in agent implementation file headers or adjacent schema files.
- **R-AGENT-006** SHOULD: FileSystemUtils SHOULD be used for consistent file reading operations before parsing configuration content.

### Verify

```bash
# Verify all parsing operations are protected with error handling
grep -r 'JSON.parse\|parseTOML' src/agents/ src/core/UnifiedConfigLoader.ts | grep -v 'try\|catch' && echo 'FAIL: Unprotected parsing found' || echo 'PASS: All parsing is protected'

# Verify agent contracts are exported
grep -r 'export class.*Agent.*implements IAgent' src/agents/*.ts | wc -l | grep -E '^[5-9]|^[1-9][0-9]+$' && echo 'PASS: Agent contracts exported' || echo 'FAIL: Missing agent exports'

# Verify concurrency pattern is applied
grep -r 'applyRulerConfig' src/agents/ | wc -l | grep -E '^[5-9]|^[1-9][0-9]+$' && echo 'PASS: Concurrency pattern applied' || echo 'FAIL: Missing applyRulerConfig'
```

**Accept when:**
- All agent implementations export IAgent-compliant public contracts with consistent naming
- Configuration parsing operations use JSON.parse() or parseTOML() with error handling before data usage
- No unvalidated configuration data reaches applyRulerConfig or business logic execution paths
- Code review confirms validation occurs at configuration load boundaries for all agents
- Error messages are sanitized to remove absolute paths and sensitive values

<enforcement>
Claude Code MUST NOT skip or defer verification. All parsing operations must be protected, all agents must export IAgent contracts, and validation must occur at configuration load boundaries. Violations block merge and trigger security team notification.
</enforcement>