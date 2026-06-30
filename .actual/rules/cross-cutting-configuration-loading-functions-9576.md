# Validate External JSON Input via JSON.parse in Agent Configuration Loading: Configuration Loading Functions

These rules are ALWAYS ACTIVE for all agent implementations that load external configuration files.

### Rules

- **R-CONFIG-001** SHOULD: Configuration loading functions (readGeminiSettings, applyRulerConfig) SHOULD fail fast when JSON.parse throws, preventing agents from initializing with invalid state

### Verify

```bash
# Check for JSON.parse usage in agent configuration loading
grep -r 'JSON\.parse.*fs\.readFile\|JSON\.parse.*await fs\.readFile' src/agents/ tests/unit/agents/

# Verify configuration loading functions contain JSON.parse
grep -r 'readGeminiSettings\|applyRulerConfig' src/agents/ --include='*.ts' -A 5 | grep 'JSON\.parse'

# Run configuration loading tests
npm test -- --testPathPattern='GeminiCliAgent|OpenCodeAgent|ZedAgent' --testNamePattern='configuration|settings'
```

**Accept when:**
- All agent implementations that load external JSON configuration files use JSON.parse on the raw file content string
- Configuration loading functions (readGeminiSettings, applyRulerConfig) contain explicit JSON.parse calls as evidenced by grep results
- Tests for GeminiCliAgent, OpenCodeAgent, and ZedAgent pass, confirming configuration loading behavior is preserved

<enforcement>
Claude Code MUST NOT skip or defer verification. Configuration loading without JSON.parse validation is a security boundary violation and must be caught during code review and CI pipeline checks.
</enforcement>