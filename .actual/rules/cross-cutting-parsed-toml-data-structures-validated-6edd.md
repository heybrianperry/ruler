# Adoption of @iarna/toml for Structured TOML Parsing: Parsed Toml Data Structures Validated Against

These rules are ALWAYS ACTIVE for modules responsible for loading, parsing, and converting TOML configuration or metadata into internal data structures, including agent definitions, configuration loaders, and protocol synchronization modules that ingest TOML input.

### Rules

- **R-TOML-001** SHOULD: Parsed TOML data structures MUST be validated against domain schemas immediately following successful parsing before being propagated into runtime state.

### Verify

```bash
# Discover the project manifest and execute the primary automated test suite to verify configuration parsing routines
# Discover and run project static analysis scripts to verify dependency imports conform to the designated parser
```

**Accept when:**
- All automated test suites covering TOML parsing across configuration, agent, and protocol synchronization pass cleanly.
- Static analysis confirms no alternate TOML parsing libraries are imported or invoked across the codebase.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>