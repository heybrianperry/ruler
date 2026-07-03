# Standardize Agent Configuration Loading with Input Validation and Contract Exports: Configuration Loaders Handle

These rules are ALWAYS ACTIVE for all agent implementations and configuration loaders within the system, including all files in `src/agents/` directory, `src/core/UnifiedConfigLoader.ts`, and related configuration loading utilities.

### Rules

- **R-CONFIG-001** MUST: Configuration loaders MUST handle parsing errors and provide meaningful error messages when validation fails.
- **R-CONFIG-002** MUST: All JSON.parse() and parseTOML() calls MUST be wrapped in try-catch blocks with agent-specific error context.
- **R-CONFIG-003** MUST: Agent classes MUST be exported as named exports matching the class name and implementing IAgent interface (e.g., `export class OpenCodeAgent implements IAgent`).
- **R-CONFIG-004** MUST: Configuration validation MUST occur immediately after parsing and before passing to applyRulerConfig or other business logic.
- **R-CONFIG-005** MUST: Error messages MUST be sanitized to remove absolute paths and sensitive values; full details logged securely for debugging.
- **R-CONFIG-006** SHOULD: Configuration schema expectations SHOULD be documented in agent implementation file headers or adjacent schema files.
- **R-CONFIG-007** SHOULD: FileSystemUtils SHOULD be used for consistent file reading operations before parsing configuration content.

### Verify

```bash
# Check for unprotected parsing operations
grep -r 'JSON.parse\|parseTOML' src/agents/ src/core/UnifiedConfigLoader.ts | grep -v 'try\|catch' && echo 'FAIL: Unprotected parsing found' || echo 'PASS: All parsing is protected'

# Verify agent contract exports
grep -r 'export class.*Agent.*implements IAgent' src/agents/*.ts | wc -l | grep -E '^[5-9]|^[1-9][0-9]+$' && echo 'PASS: Agent contracts exported' || echo 'FAIL: Missing agent exports'

# Verify concurrency pattern application
grep -r 'applyRulerConfig' src/agents/ | wc -l | grep -E '^[5-9]|^[1-9][0-9]+$' && echo 'PASS: Concurrency pattern applied' || echo 'FAIL: Missing applyRulerConfig'
```

**Accept when:**
- All agent implementations export IAgent-compliant public contracts with consistent naming
- Configuration parsing operations use JSON.parse() or parseTOML() with error handling before data usage
- No unvalidated configuration data reaches applyRulerConfig or business logic execution paths
- Code review confirms validation occurs at configuration load boundaries for all agents
- Error messages are sanitized and do not expose sensitive file paths or configuration structure

<enforcement>
Claude Code MUST NOT skip or defer verification. All parsing operations must be protected with error handling. All agent implementations must export IAgent contracts. Configuration validation must occur at load time before business logic execution.
</enforcement>