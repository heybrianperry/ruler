# @iarna/toml Library Adoption for TOML Configuration Parsing: Components Parsing Structured Configuration Validate Parsed

These rules are ALWAYS ACTIVE for all application modules, configuration loaders, agent drivers, and protocol synchronization components that read or parse TOML data.

### Rules

- **R-TOML-001** SHOULD: Components parsing structured configuration SHOULD validate the parsed object output against a strict schema definition immediately after TOML deserialization.

### Verify

```bash
# Discover and execute the project automated test runner
# Discover and execute the project linter and type checker
```

**Accept when:**
- All automated test suites covering configuration loading and TOML parsing pass without errors or regressions.
- Static type analysis and linting checks complete with zero errors across all modules importing the TOML parser.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via automated continuous integration test suites and static analysis.
</enforcement>