# Adoption of @iarna/toml for Structured TOML Parsing: Components Not Introduce Alternate Toml Parsing

These rules are ALWAYS ACTIVE for modules responsible for loading, parsing, and converting TOML configuration or metadata into internal data structures, agent definitions, configuration loaders, and protocol synchronization modules that ingest TOML input.

### Rules

- **R-TOML-001** MUST_NOT: Components MUST NOT introduce alternate TOML parsing libraries or implement ad-hoc custom TOML parsers.
- **R-TOML-002** MANDATORY: Ensure raw string inputs passed to parseTOML originate from validated filesystem reads or trusted memory buffers.
- **R-TOML-003** MANDATORY: Isolate parser invocation logic within dedicated loader routines to prevent direct parser dependency leakage into business logic.

### Verify

```bash
# Discover the project manifest and execute the primary automated test suite to verify configuration parsing routines
# Discover and run project static analysis scripts to verify dependency imports conform to the designated parser
```

**Accept when:**
- All automated test suites covering TOML parsing across configuration, agent, and protocol synchronization pass cleanly.
- Static analysis confirms no alternate TOML parsing libraries are imported or invoked across the codebase.

<enforcement>
Claude Code MUST NOT skip or defer verification. All pull requests introducing unauthorized parsing libraries or unvalidated parseTOML invocations are blocked until aligned with standards.
</enforcement>