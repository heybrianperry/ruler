# js-yaml Adoption for Configuration Serialization and Parsing: Subsystems Handling Yaml Processing Not Introduce

These rules are ALWAYS ACTIVE for all core processing modules and utility functions responsible for serializing or deserializing structured YAML configuration documents.

### Rules

- **R-YAML-001** MUST_NOT: Subsystems handling YAML processing MUST NOT introduce secondary or redundant YAML parsing libraries for configuration processing.
- **R-YAML-002** MUST: Configure parsing routines to enforce strict schema adherence and disable arbitrary runtime object instantiation.
- **R-YAML-003** MUST: Encapsulate parsing invocations within shared utility abstractions to maintain consistent deserialization options and schema validation across processing components.

### Verify

```bash
# Discover the project test runner configuration from the repository manifest and execute the test suite covering core document parsing and utility operations.
# Discover the static analysis and linting scripts from the project manifest and execute checks to verify uniform dependency imports and absence of unauthorized parsing libraries.
```

**Accept when:**
- All automated test suites covering document parsing pass successfully with valid sample configurations and reject malformed document inputs with structured errors.
- Static analysis and import verification checks confirm that only the approved serialization library is imported for YAML processing across core modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory and enforced by continuous integration pipelines and static analysis.
</enforcement>