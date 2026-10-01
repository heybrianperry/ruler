# Adoption of js-yaml for Core YAML Document Processing: Core Processing Components Validate Parsed Yaml

These rules are ALWAYS ACTIVE for all core processing and utility modules performing YAML parsing, serialization, or configuration loading.

### Rules

- **R-YAML-001** MUST: Core processing components MUST validate parsed YAML output against expected internal schemas and sanitize input contents prior to downstream execution.

### Verify

```bash
# Discover and run the project build, test suite, and linter from the manifest
# 1. Discover the repository build script from the project manifest and execute the primary build target.
# 2. Discover the automated test suite runner from the project manifest and execute unit and integration test suites covering document processing modules.
# 3. Discover the static analysis and linting script from the project manifest and run source code validation across core modules.
```

**Accept when:**
- All core modules parsing YAML documents invoke js-yaml consistently without secondary parser imports.
- Repository verification and automated test suites confirm successful parsing and validation across document processing workflows.
- Dependency lock artifacts confirm resolved library versions match project specifications.

<enforcement>
Claude Code MUST NOT skip or defer verification. All core modules parsing YAML must use the approved library, validate parsed schemas, and pass all repository verification scripts.
</enforcement>