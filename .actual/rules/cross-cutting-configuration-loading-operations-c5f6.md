# Standardize Agent Configuration Loading with Input Validation and Contract Exports: Configuration Loading Operations

These rules are ALWAYS ACTIVE for all agent implementations and configuration loaders within the system, specifically targeting files in `src/agents/` directory, `src/core/UnifiedConfigLoader.ts`, and related configuration parsing utilities.

### Rules

- **R-CONFIG-001** MUST: Configuration loading operations MUST validate input using `JSON.parse()` for JSON files or `parseTOML()` for TOML files before processing.
- **R-CONFIG-002** MUST: Wrap all `JSON.parse()` and `parseTOML()` calls in try-catch blocks with agent-specific error context.
- **R-CONFIG-003** MUST: Export agent classes as named exports matching the class name (e.g., `export class OpenCodeAgent implements IAgent`).
- **R-CONFIG-004** MUST: Validate configuration immediately after parsing and before passing to `applyRulerConfig` or other business logic.
- **R-CONFIG-005** SHOULD: Document expected configuration schema in agent implementation file headers or adjacent schema files.
- **R-CONFIG-006** SHOULD: Use FileSystemUtils for consistent file reading operations before parsing configuration content.
- **R-CONFIG-007** SHOULD: Sanitize error messages to remove absolute paths and sensitive values; log full details securely for debugging.

### Verify

```bash
# Check for unprotected JSON.parse and parseTOML calls
grep -r 'JSON.parse\|parseTOML' src/agents/ src/core/UnifiedConfigLoader.ts | grep -v 'try\|catch' && echo 'FAIL: Unprotected parsing found' || echo 'PASS: All parsing is protected'

# Verify agent contract exports
grep -r 'export class.*Agent.*implements IAgent' src/agents/*.ts | wc -l | grep -E '^[5-9]|^[1-9][0-9]+$' && echo 'PASS: Agent contracts exported' || echo 'FAIL: Missing agent exports'

# Verify applyRulerConfig concurrency pattern is applied
grep -r 'applyRulerConfig' src/agents/ | wc -l | grep -E '^[5-9]|^[1-9][0-9]+$' && echo 'PASS: Concurrency pattern applied' || echo 'FAIL: Missing applyRulerConfig'
```

**Accept when:**
- All agent implementations export IAgent-compliant public contracts with consistent naming
- Configuration parsing operations use `JSON.parse()` or `parseTOML()` with error handling before data usage
- No unvalidated configuration data reaches `applyRulerConfig` or business logic execution paths
- Code review confirms validation occurs at configuration load boundaries for all agents
- Error messages are sanitized to prevent exposure of sensitive file paths or configuration structure

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration loading operations MUST be protected with validation and error handling before merge. Violations trigger CI pipeline failure and mandatory code review blocking.
</enforcement>