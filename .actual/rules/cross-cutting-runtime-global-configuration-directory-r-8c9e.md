# XDG_CONFIG_HOME Environment Configuration Resolution: Runtime Global Configuration Directory Resolution Prioritize

These rules are ALWAYS ACTIVE for runtime initialization, command-line execution handlers, and configuration loaders resolving global user configuration directories.

### Rules

- **R-XDG-001** MUST: Runtime global configuration directory resolution MUST prioritize the process.env.XDG_CONFIG_HOME environment variable when set and non-empty before falling back to the standard operating system user configuration path.
- **R-XDG-002** MUST: Validate that process.env.XDG_CONFIG_HOME is non-empty, contains valid characters, and resolves to an absolute path before adopting it as the configuration base.
- **R-XDG-003** MUST: Encapsulate global configuration directory resolution within a single reusable utility function that checks process.env.XDG_CONFIG_HOME, applies fallback to standard user home directory paths, and normalizes the resulting path.
- **R-XDG-004** MUST: Ensure all consumer modules across execution boundaries import the shared resolution utility rather than reading process.env directly.

### Verify

```bash
# Discover test runner from manifest and run test suite verifying configuration resolution
# Discover static analysis tool from manifest and verify direct references to process.env.XDG_CONFIG_HOME are restricted
```

**Accept when:**
- Test suites pass verifying that setting process.env.XDG_CONFIG_HOME redirects global configuration discovery to the specified directory path.
- Test suites pass verifying that unsetting or clearing process.env.XDG_CONFIG_HOME correctly falls back to the default operating system user directory.
- Static analysis checks confirm that direct access to process.env.XDG_CONFIG_HOME is restricted to designated configuration resolution modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull requests containing unencapsulated direct access to process.env.XDG_CONFIG_HOME are blocked until refactored to use the central configuration utility.
</enforcement>