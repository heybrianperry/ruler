# js-yaml Adoption for YAML Processing: Core Processing Modules Parsing Serializing Yaml

These rules are ALWAYS ACTIVE for all core processing modules and utility components requiring YAML document parsing and serialization.

### Rules

- **R-YAML-001** MUST: Core processing modules parsing or serializing YAML definitions MUST use js-yaml as the standardized parser module.
- **R-YAML-002** MANDATORY: Ground all implementations against the repository resolution artifact per the Discovery Policy prior to writing code that uses a versioned library.
- **R-YAML-003** MANDATORY: Encapsulate parsing calls within dedicated utility wrappers to isolate schema validation and error transformation logic.
- **R-YAML-004** MANDATORY: Ensure asynchronous file read operations complete before passing raw document content to parser methods.
- **R-YAML-005** MANDATORY: Apply input validation checks and strict schema parsing guards on all deserialized content.

### Verify

```bash
# Discover the repository test runner from the project manifest and execute the test suite governing configuration parsing.
# Discover the linting and static analysis command from the build configuration and execute validation across core processing modules.
```

**Accept when:**
- All configuration parsing modules invoke js-yaml for YAML deserialization and pass verification suites.
- Input validation routines successfully intercept malformed YAML content prior to downstream processing.
- Dependency analysis confirms no secondary YAML parsing libraries exist within core modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated dependency linting, static code analysis, and peer code reviews enforce compliance.
</enforcement>