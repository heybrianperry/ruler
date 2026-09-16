# Use of JSON.parse() and TOML Parsing for Secure Input Validation: Implement Robust Error Handling Mechanisms Around

These rules are ALWAYS ACTIVE for code that parses external JSON and TOML configuration or data files, including modules loading configuration, agent implementations processing external data, and CLI commands consuming structured input.

### Rules

- **R-PARSE-001** SHOULD: Implement robust error handling mechanisms around all parsing operations to gracefully manage malformed or invalid input.

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