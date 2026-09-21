# Adoption of @iarna/toml for Structured TOML Parsing: Before Writing Modifying Code That Depends

These rules are ALWAYS ACTIVE for modules responsible for loading, parsing, and converting TOML configuration or metadata into internal data structures, including agent definitions, configuration loaders, and protocol synchronization modules that ingest TOML input.

### Rules

- **R-TOML-001** MUST: Before writing or modifying code that depends on @iarna/toml, consumers MUST discover the project dependency manifest and authoritative lock artifact to inspect the exact resolved library version and verify API compatibility against public reference documentation.
- **R-TOML-002** MUST: Ensure raw string inputs passed to parseTOML originate from validated filesystem reads or trusted memory buffers.
- **R-TOML-003** MUST: Isolate parser invocation logic within dedicated loader routines to prevent direct parser dependency leakage into business logic.
- **R-TOML-004** MUST: Wrap parseTOML calls in structured error-handling routines that capture parsing failures and emit descriptive diagnostics.
- **R-TOML-005** MUST: Enforce strict schema validation immediately following parser invocation before exposing configuration values.

### Verify

```bash
# Discover the project manifest and execute the primary automated test suite to verify configuration parsing routines
# Discover and run project static analysis scripts to verify dependency imports conform to the designated parser
```

**Accept when:**
- All automated test suites covering TOML parsing across configuration, agent, and protocol synchronization pass cleanly.
- Static analysis confirms no alternate TOML parsing libraries are imported or invoked across the codebase.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests introducing unauthorized parsing libraries or unvalidated parseTOML invocations are blocked until aligned with standards.
</enforcement>