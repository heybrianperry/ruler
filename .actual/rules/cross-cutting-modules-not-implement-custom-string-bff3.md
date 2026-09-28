# Adoption of @iarna/toml for TOML Configuration Parsing: Modules Not Implement Custom String Splitting

These rules are ALWAYS ACTIVE for all subsystems, loader modules, agent controllers, and engine components parsing TOML configuration files, manifests, or text blocks.

### Rules

- **R-TOML-001** MUST_NOT: Modules MUST NOT implement custom string-splitting or regular-expression-based TOML parsing logic, nor import alternative TOML parsing libraries.
- **R-TOML-002** MANDATORY: Execute the discovery policy to identify the project manifest, build tool, and resolved version in the lock file before using versioned libraries.

### Verify

```bash
# Discover and execute the project test runner to verify that all configuration and manifest parsing unit tests pass.
# Discover and execute the project static analysis and linting tools to ensure imports comply with approved dependency boundaries.
```

**Accept when:**
- All test suites exercising configuration parsing, agent definitions, and rule execution pass successfully.
- Static analysis verifies that TOML parsing imports adhere to the approved core library without unauthorized alternatives.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated continuous integration test suites, static analysis rules verifying dependency import boundaries, and architecture review gates.
</enforcement>