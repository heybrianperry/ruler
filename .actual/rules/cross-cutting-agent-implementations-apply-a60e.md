# Standardize Agent Configuration Loading with Input Validation and Contract Exports: Agent Implementations Apply

These rules are ALWAYS ACTIVE for all agent implementations and configuration loaders within the system, including all files in `src/agents/` directory and `src/core/UnifiedConfigLoader.ts`.

### Rules

- **R-AGENT-CONFIG-001** MUST: Agent implementations MUST apply the applyRulerConfig concurrency pattern with validated configuration input.
- **R-AGENT-CONFIG-002** MUST: All JSON.parse() and parseTOML() calls in agent implementations and configuration loaders MUST be wrapped in try-catch blocks with agent-specific error context.
- **R-AGENT-CONFIG-003** MUST: Agent classes MUST be exported as named exports matching the class name and implementing the IAgent interface (e.g., `export class OpenCodeAgent implements IAgent`).
- **R-AGENT-CONFIG-004** MUST: Configuration validation MUST occur immediately after parsing and before passing data to applyRulerConfig or other business logic.
- **R-AGENT-CONFIG-005** SHOULD: Configuration schema expectations SHOULD be documented in agent implementation file headers or adjacent schema files.
- **R-AGENT-CONFIG-006** SHOULD: FileSystemUtils SHOULD be used for consistent file reading operations before parsing configuration content.
- **R-AGENT-CONFIG-007** SHOULD: Error messages SHOULD be sanitized to remove absolute paths and sensitive values; full details logged securely for debugging.

### Verify

```bash
# Check that all JSON.parse and parseTOML calls are protected with error handling
grep -r 'JSON.parse\|parseTOML' src/agents/ src/core/UnifiedConfigLoader.ts | grep -v 'try\|catch' && echo 'FAIL: Unprotected parsing found' || echo 'PASS: All parsing is protected'

# Verify agent contracts are exported
grep -r 'export class.*Agent.*implements IAgent' src/agents/*.ts | wc -l | grep -E '^[5-9]|^[1-9][0-9]+$' && echo 'PASS: Agent contracts exported' || echo 'FAIL: Missing agent exports'

# Verify applyRulerConfig concurrency pattern is applied
grep -r 'applyRulerConfig' src/agents/ | wc -l | grep -E '^[5-9]|^[1-9][0-9]+$' && echo 'PASS: Concurrency pattern applied' || echo 'FAIL: Missing applyRulerConfig'
```

**Accept when:**
- All agent implementations export IAgent-compliant public contracts with consistent naming
- Configuration parsing operations use JSON.parse() or parseTOML() with error handling before data usage
- No unvalidated configuration data reaches applyRulerConfig or business logic execution paths
- Code review confirms validation occurs at configuration load boundaries for all agents
- All five agent implementations (OpenCodeAgent, MistralVibeAgent, CodexCliAgent, AmazonQCliAgent, and any additional agents) apply the applyRulerConfig pattern with validated input

<enforcement>
Claude Code MUST NOT skip or defer verification. All parsing operations must be protected, all agent contracts must be exported, and all agents must apply the concurrency pattern with validated configuration input. Violations block merge and require security team notification.
</enforcement>