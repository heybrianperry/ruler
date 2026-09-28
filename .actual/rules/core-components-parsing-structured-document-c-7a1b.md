# js-yaml Adoption for YAML Processing: Components Parsing Structured Document Content Perform

These rules are ALWAYS ACTIVE for all core processing modules and utility components requiring YAML document parsing and serialization.

### Rules

- **R-YML-001** MUST: Components parsing structured document content MUST perform input validation on deserialized objects prior to downstream processing.

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
Claude Code MUST NOT skip or defer verification. Automated dependency linting, static code analysis, and peer code review enforce compliance.
</enforcement>