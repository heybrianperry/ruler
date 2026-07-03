# Standardize Agent Configuration Loading with Input Validation and Contract Exports: Configuration Validation Occur

These rules are ALWAYS ACTIVE for all agent implementations, configuration loaders, and related utilities within the system that handle external configuration from JSON and TOML files.

### Rules

- **R-CFG-001** SHOULD: Configuration validation SHOULD occur before any configuration data is used in business logic or passed to downstream components.
- **R-CFG-002** MUST: Wrap all JSON.parse() and parseTOML() calls in try-catch blocks with agent-specific error context.
- **R-CFG-003** MUST: Export agent classes as named exports matching the class name (e.g., export class OpenCodeAgent implements IAgent).
- **R-CFG-004** MUST: Validate configuration immediately after parsing and before passing to applyRulerConfig or other business logic.
- **R-CFG-005** SHOULD: Document expected configuration schema in agent implementation file headers or adjacent schema files.
- **R-CFG-006** SHOULD: Use FileSystemUtils for consistent file reading operations before parsing configuration content.
- **R-CFG-007** SHOULD: Sanitize error messages to remove absolute paths and sensitive values; log full details securely for debugging.

### Verify

```bash
# Check for unprotected JSON.parse and parseTOML calls
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
- Error messages are sanitized to prevent exposure of sensitive file paths or configuration structure

<enforcement>
Claude Code MUST NOT skip or defer verification. All parsing operations MUST be protected with error handling. All agent implementations MUST export IAgent-compliant contracts. Configuration validation MUST occur before business logic usage.
</enforcement>