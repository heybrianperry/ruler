# js-yaml Adoption for YAML Processing: Components Not Introduce Alternative Yaml Parsing

These rules are ALWAYS ACTIVE for all core processing modules, utility components, and input validation routines requiring YAML document parsing and serialization.

### Rules

- **R-YAML-001** MUST_NOT: Components MUST_NOT introduce alternative YAML parsing libraries into core processing boundaries.

### Verify

```bash
# Discover and run the project test suite governing configuration parsing
# Discover and run the linting and static analysis command across core processing modules
```

**Accept when:**
- All configuration parsing modules invoke js-yaml for YAML deserialization and pass verification suites.
- Input validation routines successfully intercept malformed YAML content prior to downstream processing.
- Dependency analysis confirms no secondary YAML parsing libraries exist within core modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated dependency linting, static code analysis, and peer code review enforce these rules.
</enforcement>