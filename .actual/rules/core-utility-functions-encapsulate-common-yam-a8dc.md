# Adoption of js-yaml for Core YAML Document Processing: Utility Functions Encapsulate Common Yaml Parsing

These rules are ALWAYS ACTIVE for all core processing and utility modules performing YAML parsing, serialization, or configuration loading.

### Rules

- **R-YAML-001** MAY: Utility functions MAY encapsulate common js-yaml parsing options to provide uniform error handling across core modules.

### Verify

```bash
# Discover and execute the primary build target from the project manifest
# Discover and execute the automated test suite runner for document processing modules
# Discover and run the static analysis and linting script across core modules
```

**Accept when:**
- All core modules parsing YAML documents invoke js-yaml consistently without secondary parser imports.
- Repository verification and automated test suites confirm successful parsing and validation across document processing workflows.
- Dependency lock artifacts confirm resolved library versions match project specifications.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via static analysis, code reviews, and CI test passes.
</enforcement>