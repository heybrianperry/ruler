<rule_activation id="f0d5cbf0-ecef-4cff-b1f4-5e7cb204efc8" title="Adopt Unified Configuration Loading with Input Validation: Configuration Loading Failures" applies_to="**/*">
These rules are ALWAYS ACTIVE for all configuration loading operations across agent implementations, core engine components, and test suites. All configuration sources MUST be processed through the unified configuration loader with mandatory input validation.
</rule_activation>

### Rules

- **R-UCL-001** MUST: Configuration loading failures MUST result in explicit error messages that identify the validation failure without exposing sensitive system information.

### Verify

```bash
# Verify UnifiedConfigLoader adoption across codebase
grep -r 'UnifiedConfigLoader' src/ | wc -l

# Verify all agent implementations use UnifiedConfigLoader
grep -r 'new UnifiedConfigLoader\|import.*UnifiedConfigLoader' src/agents/ src/core/ src/mcp/

# Verify configuration validation tests pass
npm test -- --grep 'configuration.*validation'

# Verify no direct configuration parsing outside UnifiedConfigLoader
eslint src/ --rule 'no-restricted-imports: [error, {patterns: ["**/direct-config-parser"]}]'
```

**Accept when:**
- All agent implementations (ZedAgent, GeminiCliAgent, RooCodeAgent, OpenCodeAgent, QwenCodeAgent) import and use UnifiedConfigLoader
- Configuration validation tests pass for both valid inputs and properly reject invalid inputs with clear error messages
- No direct TOML parsing or environment variable access occurs outside of UnifiedConfigLoader in production code
- Code review confirms all new configuration loading code uses the unified approach
- Configuration loading failures produce explicit error messages identifying validation failures without exposing sensitive system information

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration loading operations MUST use UnifiedConfigLoader with mandatory input validation. Violations detected by CI pipeline checks, ESLint rules, code review, and security scanning tools will block merge requests and require documented exceptions with architecture team approval.
</enforcement>