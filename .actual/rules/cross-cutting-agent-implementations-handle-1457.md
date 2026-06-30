# Validate External JSON Input via JSON.parse in Agent Configuration Loading: Agent Implementations Handle

These rules are ALWAYS ACTIVE for all agent implementations that load external configuration files.

### Rules

- **R-AGENT-CONFIG-001** SHOULD: Agent implementations SHOULD handle JSON.parse exceptions (SyntaxError) explicitly to provide meaningful error messages during configuration loading

### Verify

```bash
# Check for JSON.parse usage in agent configuration loading
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
- JSON.parse exceptions are caught and handled with meaningful error messages that include the configuration file path

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent implementations loading external JSON configuration must validate input via JSON.parse with explicit exception handling.
</enforcement>