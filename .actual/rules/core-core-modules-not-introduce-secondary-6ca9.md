# Adoption of js-yaml for Core YAML Document Processing: Core Modules Not Introduce Secondary Redundant

These rules are ALWAYS ACTIVE for core processing and utility modules performing YAML parsing, serialization, or configuration loading.

### Rules

- **R-YAML-001** MUST_NOT: Core modules MUST NOT introduce secondary or redundant YAML parsing libraries for configuration or document decoding tasks.

### Verify

```bash
# Discover the repository build script from the project manifest and execute the primary build target.
# Discover the automated test suite runner from the project manifest and execute unit and integration test suites covering document processing modules.
# Discover the static analysis and linting script from the project manifest and run source code validation across core modules.
```

**Accept when:**
- All core modules parsing YAML documents invoke js-yaml consistently without secondary parser imports.
- Repository verification and automated test suites confirm successful parsing and validation across document processing workflows.
- Dependency lock artifacts confirm resolved library versions match project specifications.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated static analysis, linting rules, and peer code reviews verify compliance, and violations block pull requests or require refactoring.
</enforcement>