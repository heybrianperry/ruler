# formatCliError Formatted CLI Error Logging: Developers Inspect Repository Lock Artifact Determine

These rules are ALWAYS ACTIVE for all command-line interface execution handlers managing user-facing command dispatch and failure responses.

### Rules

- **R-FORMAT-001** MUST: Developers MUST inspect the repository lock artifact to determine and adhere to exact resolved dependency versions before implementing or updating logging utilities.
- **R-FORMAT-002** MUST: All command-line error handlers process failure strings through formatCliError before calling console.error.
- **R-FORMAT-003** MUST: Standard error emissions remain decoupled from standard result output pipelines.

### Verify

```bash
# Discover and run the repository linting suite to verify compliance with error formatting rules across CLI handlers
# Discover and execute the automated test suite targeting CLI error handling and terminal output streams
```

**Accept when:**
- All command-line error handlers process failure strings through formatCliError before calling console.error.
- Automated tests confirm error output is written to standard error with formatted structure.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verified by automated static analysis and code review checks during pull request evaluation.
</enforcement>