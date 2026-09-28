# formatCliError Formatted CLI Error Logging: Cli Error Handlers Not Emit Unformatted

These rules are ALWAYS ACTIVE for all command-line interface execution handlers managing user-facing command dispatch and failure responses.

### Rules

- **R-CLI-001** MUST_NOT: CLI error handlers MUST NOT emit unformatted operational error strings directly to standard error output.

### Verify

```bash
# Discover and run the repository linting suite to verify compliance with error formatting rules across CLI handlers.
# Discover and execute the automated test suite targeting CLI error handling and terminal output streams.
```

**Accept when:**
- All command-line error handlers process failure strings through formatCliError before calling console.error.
- Automated tests confirm error output is written to standard error with formatted structure.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via automated static analysis, code review checks, and automated integration tests validating terminal error emission behavior.
</enforcement>