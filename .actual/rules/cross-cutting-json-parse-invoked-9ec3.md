# Validate External JSON Input via JSON.parse in Agent Configuration Loading: Json Parse Invoked

These rules are ALWAYS ACTIVE for all agent implementations that load external configuration files, including all agent classes extending AgentsMdAgent or implementing IAgent, configuration loading functions, test harnesses, and MCP configuration loading.

### Rules

- **R-JSON-PARSE-001** MUST: JSON.parse MUST be invoked on the raw string content returned from file system read operations, not on pre-processed or partially parsed data.

### Verify

```bash
# Check for JSON.parse usage with fs.readFile in agent implementations
grep -r 'JSON\.parse.*fs\.readFile\|JSON\.parse.*await fs\.readFile' src/agents/ tests/unit/agents/

# Verify configuration loading functions use JSON.parse
grep -r 'readGeminiSettings\|applyRulerConfig' src/agents/ --include='*.ts' -A 5 | grep 'JSON\.parse'

# Run agent configuration tests
npm test -- --testPathPattern='GeminiCliAgent|OpenCodeAgent|ZedAgent' --testNamePattern='configuration|settings'
```

**Accept when:**
- All agent implementations that load external JSON configuration files use JSON.parse on the raw file content string
- Configuration loading functions (readGeminiSettings, applyRulerConfig) contain explicit JSON.parse calls as evidenced by grep results
- Tests for GeminiCliAgent, OpenCodeAgent, and ZedAgent pass, confirming configuration loading behavior is preserved
- JSON.parse is applied before any agent logic consumes the configuration object
- Error handling for JSON.parse failures (SyntaxError) is present in configuration loading code paths

<enforcement>
Claude Code MUST NOT skip or defer verification of this rule. All agent configuration loading code must be reviewed to ensure JSON.parse is applied to raw file content. Violations must be caught during code review and CI pipeline checks before merge.
</enforcement>