# Validate External JSON Input via JSON.parse in Agent Configuration Loading: Agent Implementations Apply

These rules are ALWAYS ACTIVE for all agent implementations that load external configuration files.

### Rules

- **R-AGENT-CONFIG-001** MUST: All agent implementations MUST apply JSON.parse to configuration file content loaded via fs.readFile or fs/promises before consuming the data

### Verify

```bash
# Check for JSON.parse usage with fs.readFile in agent configuration loading
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
- JSON.parse calls are wrapped in try-catch blocks to handle SyntaxError exceptions
- Configuration file paths are included in error messages for debugging

<enforcement>
Claude Code MUST NOT skip or defer verification. All agent implementations loading external JSON configuration must be verified to use JSON.parse before consuming file content. Violations must be caught in code review and CI pipeline checks.
</enforcement>