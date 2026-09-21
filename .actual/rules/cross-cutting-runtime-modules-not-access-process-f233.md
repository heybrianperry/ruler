# XDG_CONFIG_HOME Environment Configuration Resolution: Runtime Modules Not Access Process Env

These rules are ALWAYS ACTIVE for global configuration resolution, command-line execution handlers, and runtime initialization modules.

### Rules

- **R-XDG-001** MUST_NOT: Runtime modules MUST_NOT access process.env.XDG_CONFIG_HOME directly outside of dedicated configuration resolution modules to prevent divergent path resolution semantics.

### Verify

```bash
# Discover the test runner from project manifest and execute test suite
# Discover the static analysis tool from project manifest and verify no direct process.env.XDG_CONFIG_HOME access outside configuration utility
```

**Accept when:**
- Test suites pass verifying that setting process.env.XDG_CONFIG_HOME redirects global configuration discovery to the specified directory path.
- Test suites pass verifying that unsetting or clearing process.env.XDG_CONFIG_HOME correctly falls back to the default operating system user directory.
- Static analysis checks confirm that direct access to process.env.XDG_CONFIG_HOME is restricted to designated configuration resolution modules.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>