# Adoption of js-yaml for Core YAML Document Processing: Before Implementing Updating Yaml Parsing Routines

These rules are ALWAYS ACTIVE for all core processing and utility modules performing YAML parsing, serialization, or configuration loading.

### Rules

- **R-YML-001** MUST: Before implementing or updating YAML parsing routines, developers MUST inspect the repository dependency resolution lock artifact to verify the exact resolved version of js-yaml and validate that invoked APIs match that version specification.
- **R-YML-002** MANDATORY: Execute the lock-version grounding sequence before writing code that uses a versioned library: find dependency manifest, identify build tool, inspect lock/resolution artifact, look up official documentation for that exact version, confirm API existence, and re-verify at point of use for version-sensitive behavior.
- **R-YML-003** MUST: Isolate document parsing calls within utility modules to decouple core business logic from direct library API details.
- **R-YML-004** MUST: Apply structural validation immediately after document deserialization to ensure parsed objects conform to expected internal data contracts.

### Verify

```bash
# Discover and execute the project build script from the manifest
# Discover and execute automated test suites covering document processing modules
# Discover and run static analysis and linting across core modules
```

**Accept when:**
- All core modules parsing YAML documents invoke js-yaml consistently without secondary parser imports.
- Repository verification and automated test suites confirm successful parsing and validation across document processing workflows.
- Dependency lock artifacts confirm resolved library versions match project specifications.

<enforcement>
Claude Code MUST NOT skip or defer verification. Compliance is verified via automated static analysis, peer code reviews, and continuous integration test passes.
</enforcement>