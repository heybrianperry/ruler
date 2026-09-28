# Adoption of @iarna/toml for TOML Configuration Parsing: Toml Deserialization Operations Invoke Standardized Parser

These rules are ALWAYS ACTIVE for all subsystems, loader modules, agent controllers, and engine components parsing TOML configuration files, manifests, or text blocks.

### Rules

- **R-TOML-001** MUST: All TOML deserialization operations MUST invoke the standardized parser function provided by the adopted library.

### Verify

```bash
# Discover and execute the project test runner to verify that all configuration and manifest parsing unit tests pass.
# Discover and execute the project static analysis and linting tools to ensure imports comply with approved dependency boundaries.
```

**Accept when:**
- All test suites exercising configuration parsing, agent definitions, and rule execution pass successfully.
- Static analysis verifies that TOML parsing imports adhere to the approved core library without unauthorized alternatives.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>