# Adoption of @iarna/toml for TOML Configuration Parsing: Modules Parsing Toml Formatted Configuration Metadata

These rules are ALWAYS ACTIVE for modules parsing TOML-formatted configuration, metadata, or agent definitions.

### Rules

- **R-TOML-001** MUST: Modules parsing TOML-formatted configuration, metadata, or agent definitions MUST adopt @iarna/toml as the standard TOML parsing library.

### Verify

```bash
# Discover and execute the project test runner to verify configuration and manifest parsing unit tests pass
# Discover and execute static analysis and linting tools to ensure imports comply with approved dependency boundaries
```

**Accept when:**
- All test suites exercising configuration parsing, agent definitions, and rule execution pass successfully.
- Static analysis verifies that TOML parsing imports adhere to the approved core library without unauthorized alternatives.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated continuous integration test suites, static analysis rule checks, and code reviews enforce these requirements.
</enforcement>