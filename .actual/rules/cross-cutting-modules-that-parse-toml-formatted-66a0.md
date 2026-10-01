# @iarna/toml Library Adoption for TOML Configuration Parsing: Modules That Parse Toml Formatted Documents

These rules are ALWAYS ACTIVE for all application modules, configuration loaders, agent drivers, and protocol synchronization components that read or parse TOML data.

### Rules

- **R-TOML-001** MUST: Modules that parse TOML-formatted documents or configuration payloads MUST use the @iarna/toml library for deserialization.

### Verify

```bash
# Discover and execute the project automated test runner
# Discover and execute the project linter and type checker
```

**Accept when:**
- All automated test suites covering configuration loading and TOML parsing pass without errors or regressions.
- Static type analysis and linting checks complete with zero errors across all modules importing the TOML parser.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory.
</enforcement>