# Use of JSON.parse() and TOML Parsing for Secure Input Validation: Use Parsetoml When Deserializing Toml Content

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-ADR-001** MUST: MUST use parseTOML() when deserializing TOML content from external sources.

### Verify

```bash
Inspect the project's dependency manifest to identify parsing libraries.
Examine relevant agent, CLI, and core configuration files for usage of JSON.parse() and parseTOML().
Run unit and integration tests that cover configuration loading and external data processing.
```

**Accept when:**
- All external JSON and TOML data is successfully parsed without runtime errors.
- Malformed input consistently results in controlled error handling, not application crashes.
- No direct usage of alternative parsing methods for JSON or TOML is found in new code.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>