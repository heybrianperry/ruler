# Standardize console-based logging for warnings and errors across core modules: Error Log Messages

These rules are ALWAYS ACTIVE for all modules in the Frameworks & Libraries > Libraries & Modules category that perform observability logging operations, including ConfigLoader, FileSystemUtils, FirebenderAgent, and CLI command handlers.

### Rules

- **R-CONSOLE-LOG-001** SHOULD: Error log messages SHOULD include a module-specific prefix (e.g., '[ruler]', ERROR_PREFIX) to facilitate log filtering and identification.

### Verify

```bash
# Count console.error usage in core modules
grep -r 'console\.error' src/ | grep -E '(ConfigLoader|FileSystemUtils|FirebenderAgent|handlers)' | wc -l

# Count console.warn usage in core modules
grep -r 'console\.warn' src/ | grep -E '(ConfigLoader|FileSystemUtils|FirebenderAgent|handlers)' | wc -l

# Verify no external logging frameworks are imported in core modules
! grep -r 'import.*winston\|import.*pino\|import.*bunyan' src/core/ src/agents/
```

**Accept when:**
- All error conditions in ConfigLoader, FileSystemUtils, FirebenderAgent, and handlers modules are logged using console.error with descriptive context (operation name, affected resource, error message).
- All warning conditions use console.warn with sufficient context to understand the non-fatal issue.
- No external logging framework dependencies (winston, pino, bunyan, etc.) are introduced in core library modules (src/core/, src/agents/).
- Log messages include module-specific prefixes to enable filtering and identification in multi-module applications.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All error and warning logging in scope modules must be reviewed for console method usage, appropriate context inclusion, and absence of external logging framework dependencies.
</enforcement>