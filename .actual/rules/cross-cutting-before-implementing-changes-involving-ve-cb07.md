# XDG_CONFIG_HOME Environment Configuration Resolution: Before Implementing Changes Involving Versioned Configuration

These rules are ALWAYS ACTIVE for resolution of global user configuration directories during runtime initialization and command-line execution handlers querying environment settings.

### Rules

- **R-XDG-001** MUST: Before implementing changes involving versioned configuration parsing or schema validation libraries, the consumer MUST inspect the repository lock artifact to determine the exact resolved dependency versions and verify API compatibility against official documentation.
- **R-XDG-002** MUST: Encapsulate global configuration directory resolution within a single reusable utility function that checks process.env.XDG_CONFIG_HOME, applies fallback to standard user home directory paths, and normalizes the resulting path.
- **R-XDG-003** MUST: Ensure all consumer modules across execution boundaries import the shared resolution utility rather than reading process.env directly.
- **R-XDG-004** MUST: Validate that process.env.XDG_CONFIG_HOME is non-empty, contains valid characters, and resolves to an absolute path before adopting it as the configuration base.
- **R-XDG-005** MUST: Implement structured error handling and logging around directory checks to report accessibility issues without unhandled process termination.

### Verify

```bash
# Discover test runner and static analysis tool from project repository manifest
# Run test suite verifying configuration resolution against custom and fallback environment variables
# Run static analysis to verify direct references to process.env.XDG_CONFIG_HOME are restricted to designated configuration utility modules
```

**Accept when:**
- Test suites pass verifying that setting process.env.XDG_CONFIG_HOME redirects global configuration discovery to the specified directory path.
- Test suites pass verifying that unsetting or clearing process.env.XDG_CONFIG_HOME correctly falls back to the default operating system user directory.
- Static analysis checks confirm that direct access to process.env.XDG_CONFIG_HOME is restricted to designated configuration resolution modules.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>