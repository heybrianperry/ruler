# js-yaml Adoption for Configuration Serialization and Parsing: Parsing Routines Isolate Deserialization Exceptions Within

These rules are ALWAYS ACTIVE for all core subagent processing modules and utility functions responsible for serializing or deserializing structured YAML configuration documents.

### Rules

- **R-PAR-001** SHOULD: Parsing routines SHOULD isolate deserialization exceptions within bounded error handlers that provide descriptive diagnostics without leaking raw execution stack traces.
- **R-PAR-002** MANDATORY: Configure parsing routines to enforce strict schema adherence and disable arbitrary runtime object instantiation.
- **R-PAR-003** MANDATORY: Encapsulate parsing invocations within shared utility abstractions to maintain consistent deserialization options and schema validation across processing components.
- **R-PAR-004** MANDATORY: Prior to writing code using a versioned library, discover the manifest, identify the build tool, inspect the repository lock/resolution artifact for the exact resolved version, look up official version-specific documentation, and confirm API existence.

### Verify

```bash
# Discover the project test runner configuration from the repository manifest and execute tests covering core document parsing and utility operations
# Discover static analysis and linting scripts from the project manifest and execute checks to verify uniform dependency imports and absence of unauthorized parsing libraries
```

**Accept when:**
- All automated test suites covering document parsing pass successfully with valid sample configurations and reject malformed document inputs with structured errors.
- Static analysis and import verification checks confirm that only the approved serialization library is imported for YAML processing across core modules.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>