# Adoption of @iarna/toml for Structured TOML Parsing: Components Requiring Deserialization Toml Content Use

These rules are ALWAYS ACTIVE for modules responsible for loading, parsing, and converting TOML configuration or metadata into internal data structures, including agent definitions, configuration loaders, and protocol synchronization modules that ingest TOML input.

### Rules

- **R-TOML-001** MUST: All components requiring deserialization of TOML content MUST use @iarna/toml as the designated TOML parsing library.

### Verify

```bash
# Discover the project manifest and execute the primary automated test suite to verify configuration parsing routines
# Discover and run project static analysis scripts to verify dependency imports conform to the designated parser
```

**Accept when:**
- All automated test suites covering TOML parsing across configuration, agent, and protocol synchronization pass cleanly.
- Static analysis confirms no alternate TOML parsing libraries are imported or invoked across the codebase.

<enforcement>
Claude Code MUST NOT skip or defer verification. Violation handling: Pull requests introducing unauthorized parsing libraries or unvalidated parseTOML invocations are blocked until aligned with standards.
</enforcement>