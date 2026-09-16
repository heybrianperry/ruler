# Use of JSON.parse() and TOML Parsing for Secure Input Validation: Use Json Parse When Deserializing Content

These rules are ALWAYS ACTIVE for modules responsible for loading configuration files, agent implementations that process external data, and CLI commands that consume structured input from files.

### Rules

- **R-ADR-001** MUST: Use JSON.parse() when deserializing JSON content from external sources.

### Verify

```bash
# Inspect the project's dependency manifest to identify parsing libraries.
# Examine relevant agent, CLI, and core configuration files for usage of JSON.parse() and parseTOML().
# Run unit and integration tests that cover configuration loading and external data processing.
```

**Accept when:**
- All external JSON and TOML data is successfully parsed without runtime errors.
- Malformed input consistently results in controlled error handling, not application crashes.
- No direct usage of alternative parsing methods for JSON or TOML is found in new code.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>