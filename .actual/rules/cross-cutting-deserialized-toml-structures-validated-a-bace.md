# Adoption of @iarna/toml for TOML Configuration Parsing: Deserialized Toml Structures Validated Against Domain

These rules are ALWAYS ACTIVE for all subsystems, loader modules, agent controllers, and engine components parsing TOML configuration files, manifests, or text blocks.

### Rules

- **R-TOML-001** SHOULD: Deserialized TOML structures SHOULD be validated against domain schema contracts immediately following the parsing operation.
- **R-TOML-002** MANDATORY: The consumer MUST discover tool names, file names, commands, package managers, and version numbers from the project repository.
- **R-TOML-003** MANDATORY: Before writing code that uses a versioned library, execute in order: (1) Find dependency manifest, (2) Identify build tool, (3) Inspect repository lock/resolution artifact, (4) Look up official documentation for that exact version, (5) Confirm every API/class/function exists in that version, (6) Re-run steps 3-5 per dependency at point of use.
- **R-TOML-004** MANDATORY: Encapsulate TOML deserialization calls within centralized loader utilities to normalize exceptions and decouple downstream consumers from parser internals.
- **R-TOML-005** MANDATORY: Catch syntax and parsing errors raised during deserialization and wrap them in domain-specific configuration errors with actionable diagnostics.

### Verify

```bash
# Discover and execute the project test runner to verify configuration and manifest parsing unit tests
# Discover and execute the project static analysis and linting tools to ensure imports comply with approved dependency boundaries
```

**Accept when:**
- All test suites exercising configuration parsing, agent definitions, and rule execution pass successfully.
- Static analysis verifies that TOML parsing imports adhere to the approved core library without unauthorized alternatives.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>