# js-yaml Adoption for Configuration Serialization and Parsing: Modules Parsing External Untrusted Yaml Input

These rules are ALWAYS ACTIVE for core processing modules and utility functions responsible for serializing or deserializing structured YAML configuration documents.

### Rules

- **R-YAML-001** MUST: Modules parsing external or untrusted YAML input MUST use safe parsing methods and validate parsed output structures against expected schema types prior to downstream ingestion.

### Verify

```bash
# Discover and run test suite covering core document parsing and utility operations
# Discover and run static analysis/linting scripts to verify uniform dependency imports
```

**Accept when:**
- All automated test suites covering document parsing pass successfully with valid sample configurations and reject malformed document inputs with structured errors.
- Static analysis and import verification checks confirm that only the approved serialization library is imported for YAML processing across core modules.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>