# Standardize applyRulerConfig as Configuration Application Pattern: Agent Implementations Extend

These rules are ALWAYS ACTIVE for all agent implementations and configuration loading modules within the Ruler system, including all files in `src/agents/` directory, configuration loading modules (UnifiedConfigLoader, ConfigLoader), reversion operations in revert-engine.ts, and agent utility functions that manipulate configuration state.

### Rules

- **R-APPLYRULER-001** MUST: Agent implementations extend the base applyRulerConfig pattern with agent-specific configuration logic while maintaining the core interface contract.
- **R-APPLYRULER-002** MUST: All agent implementation files in src/agents/ contain applyRulerConfig method or function definition.
- **R-APPLYRULER-003** MUST: Configuration parsing operations (JSON.parse, parseTOML) wrap parse operations in try-catch blocks with meaningful error messages.
- **R-APPLYRULER-004** MUST: Agent implementations import from './IAgent' or './AgentsMdAgent' to maintain interface contract compliance.
- **R-APPLYRULER-005** MUST: File system operations use FileSystemUtils or fs/promises to maintain compatibility with revert-engine backup and restore operations.
- **R-APPLYRULER-006** SHOULD: New agent implementations coordinate with UnifiedConfigLoader by accepting configuration objects that have already been processed through hash computation and rule processing pipelines.
- **R-APPLYRULER-007** SHOULD: Agent-specific configuration extensions be documented in module headers and ensure they do not break the core applyRulerConfig contract.

### Verify

```bash
# Count applyRulerConfig occurrences in agent implementations
grep -r 'applyRulerConfig' src/agents/ --include='*.ts' | wc -l

# Verify configuration parsing is present in applyRulerConfig implementations
grep -r 'JSON\.parse\|parseTOML' src/agents/ --include='*.ts' -A 2 | grep -c 'applyRulerConfig'

# Verify IAgent interface imports
grep -r "from.*['\"]\./(IAgent|AgentsMdAgent)" src/agents/ --include='*.ts' | wc -l

# Verify try-catch wrapping around parse operations
grep -r 'try\|catch' src/agents/ --include='*.ts' -B 2 | grep -c 'JSON\.parse\|parseTOML'
```

**Accept when:**
- All agent implementation files in src/agents/ contain applyRulerConfig method or function definition
- Configuration parsing operations (JSON.parse, parseTOML) are present in agent files that implement applyRulerConfig
- Agent implementations import from './IAgent' or './AgentsMdAgent' to maintain interface contract compliance
- Parse operations are wrapped in try-catch blocks with error handling
- File system operations use FileSystemUtils or fs/promises

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail if new agent implementations lack applyRulerConfig method. Code review MUST block merge if agent does not import from IAgent or AgentsMdAgent base classes. Runtime warnings MUST be logged when configuration application bypasses standard validation patterns.
</enforcement>