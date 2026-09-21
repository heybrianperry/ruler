# XDG_CONFIG_HOME Environment Configuration Resolution: Configuration Loaders Filesystem Utilities Handle Errors

These rules are ALWAYS ACTIVE for configuration loaders, command-line execution handlers, and filesystem utilities resolving global user configuration directories.

### Rules

- **R-XDG-001** SHOULD: Configuration loaders and filesystem utilities SHOULD handle errors during directory inspection gracefully, emitting contextual diagnostic messages when the resolved global configuration directory cannot be accessed.

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
Claude Code MUST NOT skip or defer verification. Continuous integration test suites and automated static analysis verify configuration resolution and encapsulation of environment variable access.
</enforcement>