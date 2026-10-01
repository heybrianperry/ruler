# @iarna/toml Library Adoption for TOML Configuration Parsing: Engineers Implementing Modifying Code That Integrates

These rules are ALWAYS ACTIVE for all application modules, configuration loaders, agent drivers, and protocol synchronization components that read or parse TOML data.

### Rules

- **R-IARNA-001** MUST: Engineers implementing or modifying code that integrates with this dependency MUST inspect the repository dependency manifest and authoritative lock artifact to verify the exact resolved version before using its API surface.
- **R-IARNA-002** MUST: Encapsulate TOML parsing within error-handling boundaries and validate all parsed structures against declarative schemas, ensuring parsing exceptions cleanly bubble descriptive syntax errors with line and column information when available.

### Verify

```bash
# Discover and execute project test runner, linter, and type checker to verify TOML parsing calls and tests
```

**Accept when:**
- All automated test suites covering configuration loading and TOML parsing pass without errors or regressions.
- Static type analysis and linting checks complete with zero errors across all modules importing the TOML parser.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>