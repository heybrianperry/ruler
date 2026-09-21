# js-yaml Adoption for Configuration Serialization and Parsing: Engineers Inspect Repository Dependency Resolution Artifact

These rules are ALWAYS ACTIVE for core processing modules and utility functions responsible for serializing or deserializing structured YAML configuration documents.

### Rules

- **R-YAML-001** MUST: Engineers MUST inspect the repository dependency resolution artifact to determine the exact locked version of the library and confirm compatibility before invoking parsing APIs.

### Verify

```bash
# Discover and execute test suite covering core document parsing and utility operations
# Discover and execute static analysis and linting scripts to verify uniform dependency imports and absence of unauthorized parsing libraries
```

**Accept when:**
- All automated test suites covering document parsing pass successfully with valid sample configurations and reject malformed document inputs with structured errors.
- Static analysis and import verification checks confirm that only the approved serialization library is imported for YAML processing across core modules.

<enforcement>
Verification is mandatory. Claude Code MUST NOT skip or defer verification.
</enforcement>