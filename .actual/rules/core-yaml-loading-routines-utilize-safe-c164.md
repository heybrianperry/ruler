# Adoption of js-yaml for Core YAML Document Processing: Yaml Loading Routines Utilize Safe Functions

These rules are ALWAYS ACTIVE for core processing and utility modules performing YAML parsing, serialization, or configuration loading.

### Rules

- **R-YAML-001** SHOULD: YAML loading routines SHOULD utilize safe loading functions that restrict arbitrary object instantiation during document deserialization.

### Verify

```bash
# Discover and execute the project build script, test runner, and linter from the project manifest
# Example discovery check:
npm run build
npm test
npm run lint
```

**Accept when:**
- All core modules parsing YAML documents invoke js-yaml consistently without secondary parser imports.
- Repository verification and automated test suites confirm successful parsing and validation across document processing workflows.
- Dependency lock artifacts confirm resolved library versions match project specifications.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>