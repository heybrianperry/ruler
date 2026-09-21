# XDG_CONFIG_HOME Environment Configuration Resolution: Configuration Subsystem Support Custom Override Flags

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-XDG-001** MAY: The configuration subsystem MAY support custom override flags or localized configuration scopes that bypass global configuration discovery when explicitly requested.
- **R-XDG-002** MANDATORY: Encapsulate global configuration directory resolution within a single reusable utility function that checks process.env.XDG_CONFIG_HOME, applies fallback to standard user home directory paths, and normalizes the resulting path.
- **R-XDG-003** MANDATORY: Ensure all consumer modules across execution boundaries import the shared resolution utility rather than reading process.env directly.
- **R-XDG-004** MANDATORY: Validate that process.env.XDG_CONFIG_HOME is non-empty, contains valid characters, and resolves to an absolute path before adopting it as the configuration base.

### Verify

```bash
# Discover the test runner from the project repository manifest and run test suite verifying configuration resolution
# Discover the static analysis tool from the project repository manifest and verify direct references to process.env.XDG_CONFIG_HOME are restricted
```

**Accept when:**
- Test suites pass verifying that setting process.env.XDG_CONFIG_HOME redirects global configuration discovery to the specified directory path.
- Test suites pass verifying that unsetting or clearing process.env.XDG_CONFIG_HOME correctly falls back to the default operating system user directory.
- Static analysis checks confirm that direct access to process.env.XDG_CONFIG_HOME is restricted to designated configuration resolution modules.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>