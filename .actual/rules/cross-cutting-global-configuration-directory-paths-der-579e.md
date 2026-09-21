# XDG_CONFIG_HOME Environment Configuration Resolution: Global Configuration Directory Paths Derived Process

These rules are ALWAYS ACTIVE for global configuration resolution across command-line execution, filesystem operations, and configuration loading.

### Rules

- **R-XDG-001** MUST: Global configuration directory paths derived from process.env.XDG_CONFIG_HOME MUST be resolved and normalized as absolute filesystem paths before accessing or checking directory existence.

### Verify

```bash
# Discover the test runner from the project repository manifest and run the test suite verifying configuration resolution against custom and fallback environment variables.
# Discover the static analysis tool from the project repository manifest and verify that direct references to process.env.XDG_CONFIG_HOME are restricted to designated configuration utility modules.
```

**Accept when:**
- Test suites pass verifying that setting process.env.XDG_CONFIG_HOME redirects global configuration discovery to the specified directory path.
- Test suites pass verifying that unsetting or clearing process.env.XDG_CONFIG_HOME correctly falls back to the default operating system user directory.
- Static analysis checks confirm that direct access to process.env.XDG_CONFIG_HOME is restricted to designated configuration resolution modules.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>