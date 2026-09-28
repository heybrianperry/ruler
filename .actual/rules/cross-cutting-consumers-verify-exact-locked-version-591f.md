# Adoption of @iarna/toml for TOML Configuration Parsing: Consumers Verify Exact Locked Version Library

These rules are ALWAYS ACTIVE for all subsystems, loader modules, agent controllers, and engine components parsing TOML configuration files, manifests, or text blocks.

### Rules

- **R-TOML-001** MUST: Consumers MUST verify the exact locked version of the library from the repository dependency lock artifact prior to implementation.

### Verify

```bash
# Discover and execute the project test runner to verify configuration and manifest parsing unit tests
# Discover and execute static analysis and linting tools to ensure imports comply with approved dependency boundaries
```

**Accept when:**
- All test suites exercising configuration parsing, agent definitions, and rule execution pass successfully.
- Static analysis verifies that TOML parsing imports adhere to the approved core library without unauthorized alternatives.

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration loading and agent processing workflows must be verified via automated test suites and static analysis rules.
</enforcement>