# formatCliError Formatted CLI Error Logging: Cli Handlers Normalize Failure Strings Via

These rules are ALWAYS ACTIVE for all command-line interface execution handlers managing user-facing command dispatch and failure responses.

### Rules

- **R-CLI-001** SHOULD: CLI handlers SHOULD normalize all failure strings via formatCliError before returning exit codes or completing error transitions.

### Verify

```bash
# Discover and run the repository linting suite to verify compliance with error formatting rules across CLI handlers.
# Discover and execute the automated test suite targeting CLI error handling and terminal output streams.
```

**Accept when:**
- All command-line error handlers process failure strings through formatCliError before calling console.error.
- Automated tests confirm error output is written to standard error with formatted structure.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory via automated static analysis, code review, and integration tests during pull request evaluation.
</enforcement>