# js-yaml Adoption for Configuration Serialization and Parsing: System Standardize Yaml Canonical Library Parsing

These rules are ALWAYS ACTIVE for core processing modules and utility functions responsible for serializing or deserializing structured YAML configuration documents.

### Rules

- **R-YML-001** MUST: Standardize on `js-yaml` as the canonical library for parsing and serializing structured YAML configurations within core processing modules.
- **R-YML-002** MANDATORY: Execute lock-version grounding before writing code using a versioned library (find manifest, identify build tool, inspect lock/resolution artifact, look up official documentation for exact version, confirm APIs exist, re-run at point of use).
- **R-YML-003** MUST: Configure parsing routines to enforce strict schema adherence and disable arbitrary runtime object instantiation.
- **R-YML-004** MUST: Encapsulate parsing invocations within shared utility abstractions to maintain consistent deserialization options and schema validation across processing components.

### Verify

```bash
# Discover and run the project test runner configuration covering core document parsing and utility operations
# Discover and run static analysis and linting scripts to verify uniform dependency imports and absence of unauthorized parsing libraries
```

**Accept when:**
- All automated test suites covering document parsing pass successfully with valid sample configurations and reject malformed document inputs with structured errors.
- Static analysis and import verification checks confirm that only the approved serialization library is imported for YAML processing across core modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated unit and integration test suites, static analysis, and peer reviews verify compliance. CI checks fail when unapproved parsing libraries or direct unsafe parsing calls are identified.
</enforcement>