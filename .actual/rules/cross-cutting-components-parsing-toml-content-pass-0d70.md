# Adoption of @iarna/toml for Structured TOML Parsing: Components Parsing Toml Content Pass Input

These rules are ALWAYS ACTIVE for modules responsible for loading, parsing, and converting TOML configuration or metadata into internal data structures, including agent definitions, configuration loaders, and protocol synchronization modules that ingest TOML input.

### Rules

- **R-TOML-001** MUST: Components parsing TOML content MUST pass input strings directly to the parseTOML function and handle parsing exceptions at the ingestion boundary.

### Verify

```bash
# Discover the project manifest and execute the primary automated test suite to verify configuration parsing routines
# Discover and run project static analysis scripts to verify dependency imports conform to the designated parser
```

**Accept when:**
- All automated test suites covering TOML parsing across configuration, agent, and protocol synchronization pass cleanly.
- Static analysis confirms no alternate TOML parsing libraries are imported or invoked across the codebase.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory.
</enforcement>