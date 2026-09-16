# Adoption of `js-yaml` for YAML Configuration Parsing in Core Modules: Before Implementing Any Yaml Parsing Logic

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- MUST: Before implementing any YAML parsing logic using `js-yaml`, developers MUST discover the ecosystem's lock file and resolve the exact locked version.

### Verify

```bash
# Inspect the project's dependency manifest to identify the declared version of `js-yaml`.
# Examine the project's lock file to confirm the exact resolved version of `js-yaml`.
# Review relevant source files (e.g., those in `src/core/`) to confirm the usage of `js-yaml` for YAML parsing and the presence of input validation.
```

**Accept when:**
- The project's dependency manifest explicitly lists `js-yaml`.
- The lock file confirms a specific, resolved version of `js-yaml`.
- Code reviews confirm that core modules use `js-yaml` for YAML parsing and include input validation.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>