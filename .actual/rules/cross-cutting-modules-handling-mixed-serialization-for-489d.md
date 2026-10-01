# @iarna/toml Library Adoption for TOML Configuration Parsing: Modules Handling Mixed Serialization Formats Accept

These rules are ALWAYS ACTIVE for all application modules, configuration loaders, agent drivers, and protocol synchronization components that read or parse TOML data.

### Rules

- **R-TOML-001** MAY: Modules handling mixed serialization formats MAY accept raw text input and delegate conditionally to the TOML parser based on format detection.
- **R-TOML-002** MANDATORY: The consumer MUST discover tool names, file names, commands, package managers, and version numbers from the project repository.
- **R-TOML-003** MANDATORY: Before writing code that uses a versioned library, execute the lock-version grounding process in order (find dependency manifest, identify build tool, inspect repository lock or resolution artifact, look up official documentation for that exact version, confirm APIs exist, re-run per dependency at point of use).
- **R-TOML-004** MUST: Verify that all calls to TOML parsing functions cleanly handle parsing exceptions and bubble descriptive syntax errors with line and column information when available.
- **R-TOML-005** MUST: Couple TOML parsing directly with type-safe schema validation to ensure deserialized data matches expected runtime models.

### Verify

```bash
# Discover and execute the project automated test runner
# (e.g., npm test or equivalent discovered from the repository manifest)

# Discover and execute the project linter and type checker
# (e.g., npm run lint / npx tsc or equivalent discovered from the repository manifest)
```

**Accept when:**
- All automated test suites covering configuration loading and TOML parsing pass without errors or regressions.
- Static type analysis and linting checks complete with zero errors across all modules importing the TOML parser.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated continuous integration test suites, static analysis, and peer code reviews ensuring no unapproved parsing libraries or custom parsers are introduced.
</enforcement>