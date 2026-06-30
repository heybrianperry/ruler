# Validate External JSON Input via JSON.parse in Agent Configuration Loading: Implementations Apply Additional

These rules are ALWAYS ACTIVE for all agent implementations that load external configuration files, including all agent classes extending AgentsMdAgent or implementing IAgent, configuration loading functions (readGeminiSettings, applyRulerConfig), test harnesses that load agent settings from JSON files, and MCP configuration loading in agent implementations.

### Rules

- **R-CONFIG-001** MUST: Apply JSON.parse to raw file content strings when loading external JSON configuration files to establish a validation boundary that prevents syntactically invalid data from entering agent state.
- **R-CONFIG-002** MUST: Wrap JSON.parse calls in try-catch blocks to handle SyntaxError exceptions and provide context about which configuration file failed to parse.
- **R-CONFIG-003** SHOULD: Use async/await with fs/promises for configuration loading to maintain consistency with established patterns in agent implementations.
- **R-CONFIG-004** MAY: Implementations MAY apply additional schema validation after JSON.parse to enforce structural constraints beyond syntactic correctness (e.g., using TypeScript type guards or validation libraries for critical configuration fields).
- **R-CONFIG-005** SHOULD: Consider creating a shared utility function (e.g., loadJsonConfig) that encapsulates the fs.readFile + JSON.parse pattern with standardized error handling across all agents.
- **R-CONFIG-006** SHOULD: Log the configuration file path in error messages to aid debugging when JSON.parse fails in production environments.

### Verify

```bash
# Verify JSON.parse usage in agent configuration loading
grep -r 'JSON\.parse.*fs\.readFile\|JSON\.parse.*await fs\.readFile' src/agents/ tests/unit/agents/

# Verify configuration loading functions contain JSON.parse
grep -r 'readGeminiSettings\|applyRulerConfig' src/agents/ --include='*.ts' -A 5 | grep 'JSON\.parse'

# Run configuration loading tests for agent implementations
npm test -- --testPathPattern='GeminiCliAgent|OpenCodeAgent|ZedAgent' --testNamePattern='configuration|settings'
```

**Accept when:**
- All agent implementations that load external JSON configuration files use JSON.parse on the raw file content string
- Configuration loading functions (readGeminiSettings, applyRulerConfig) contain explicit JSON.parse calls as evidenced by grep results
- Tests for GeminiCliAgent, OpenCodeAgent, and ZedAgent pass, confirming configuration loading behavior is preserved
- JSON.parse calls are wrapped in try-catch blocks with appropriate error handling
- Configuration file paths are included in error messages for debugging

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent implementations loading external JSON configuration must use JSON.parse validation. Code review must enforce this rule for all new agent configuration loading code. CI pipeline must fail if new agent implementations load JSON configuration without JSON.parse validation.
</enforcement>