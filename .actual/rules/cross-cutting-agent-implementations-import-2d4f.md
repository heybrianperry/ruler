# Standardize Agent Configuration Loading with Input Validation and Contract Exports: Agent Implementations Import

These rules are ALWAYS ACTIVE for all agent implementations in `src/agents/` directory, configuration loading logic in `UnifiedConfigLoader` and related utilities, JSON and TOML configuration file parsing operations, and public API contract exports for agent interfaces.

### Rules

- **R-AGENT-001** SHOULD: Agent implementations SHOULD import core dependencies (path, fs, fs/promises) through explicit module declarations.
- **R-AGENT-002** MUST: Wrap all JSON.parse() and parseTOML() calls in try-catch blocks with agent-specific error context.
- **R-AGENT-003** MUST: Export agent classes as named exports matching the class name (e.g., `export class OpenCodeAgent implements IAgent`).
- **R-AGENT-004** MUST: Validate configuration immediately after parsing and before passing to applyRulerConfig or other business logic.
- **R-AGENT-005** SHOULD: Document expected configuration schema in agent implementation file headers or adjacent schema files.
- **R-AGENT-006** SHOULD: Use FileSystemUtils for consistent file reading operations before parsing configuration content.
- **R-AGENT-007** MUST: Sanitize error messages to remove absolute paths and sensitive values; log full details securely for debugging.

### Verify

```bash
# Check for unprotected JSON.parse and parseTOML operations
grep -r 'JSON.parse\|parseTOML' src/agents/ src/core/UnifiedConfigLoader.ts | grep -v 'try\|catch' && echo 'FAIL: Unprotected parsing found' || echo 'PASS: All parsing is protected'

# Verify agent contract exports
grep -r 'export class.*Agent.*implements IAgent' src/agents/*.ts | wc -l | grep -E '^[5-9]|^[1-9][0-9]+$' && echo 'PASS: Agent contracts exported' || echo 'FAIL: Missing agent exports'

# Verify applyRulerConfig concurrency pattern is applied
grep -r 'applyRulerConfig' src/agents/ | wc -l | grep -E '^[5-9]|^[1-9][0-9]+$' && echo 'PASS: Concurrency pattern applied' || echo 'FAIL: Missing applyRulerConfig'
```

**Accept when:**
- All agent implementations export IAgent-compliant public contracts with consistent naming
- Configuration parsing operations use JSON.parse() or parseTOML() with error handling before data usage
- No unvalidated configuration data reaches applyRulerConfig or business logic execution paths
- Code review confirms validation occurs at configuration load boundaries for all agents
- Error messages are sanitized to remove absolute paths and sensitive values

<enforcement>
Claude Code MUST NOT skip or defer verification. All parsing operations MUST be protected with try-catch blocks. All agent implementations MUST export IAgent-compliant contracts. Configuration validation MUST occur at load time before business logic execution.
</enforcement>