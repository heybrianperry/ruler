# Adopt @iarna/toml and js-yaml for Structured Configuration Parsing: Toml Parsing Operations Invoke Parse Function

These rules are ALWAYS ACTIVE for all configuration loading subsystems, agent definition loaders, subagent processors, and any component in src/core or src/agents that requires structured data parsing from TOML or YAML files.

### Rules

- **R-TOML-001** MUST: TOML parsing operations MUST invoke the parse function from @iarna/toml with file content as input.
- **R-TOML-002** MUST: Wrap all parsing operations in try-catch blocks and provide context-rich error messages that include file path, format type, and line number if available from parser.
- **R-TOML-003** MUST: Validate parsed configuration objects against expected schema using type guards or schema validation before passing to consuming components.
- **R-TOML-004** SHOULD: Implement a configuration loader abstraction that encapsulates format detection and parser selection, isolating direct parser dependencies to a single module.
- **R-TOML-005** MUST: Ensure parsing library dependencies (@iarna/toml and js-yaml) are declared in the project's dependency manifest.

### Verify

```bash
# Discover the project's dependency manifest and verify that both parsing libraries are declared as dependencies
grep -E '(@iarna/toml|js-yaml)' package.json || echo "ERROR: Parsing libraries not found in package.json"

# Discover the project's lock artifact and confirm exact resolved versions are recorded
if [ -f package-lock.json ]; then
  grep -E '(@iarna/toml|js-yaml)' package-lock.json || echo "ERROR: Parsing libraries not found in lock file"
elif [ -f yarn.lock ]; then
  grep -E '(@iarna/toml|js-yaml)' yarn.lock || echo "ERROR: Parsing libraries not found in lock file"
fi

# Discover and execute the project's test suite to verify parsing behavior across all configuration loading subsystems
npm test -- --testPathPattern="(config|parser|toml|yaml)" || echo "WARNING: Configuration parsing tests not found or failed"

# Verify parsing operations follow error handling patterns
grep -r "try.*catch" src/core src/agents --include="*.ts" | grep -E "(toml|yaml|parse)" || echo "WARNING: Error handling patterns not found in parsing code"
```

**Accept when:**
- Dependency manifest declares both @iarna/toml and js-yaml as dependencies
- Lock artifact records exact resolved versions for both parsing libraries
- All test suites pass, confirming parsing operations work correctly across configuration loading subsystems
- Code inspection confirms parsing operations follow error handling and validation patterns specified in rules
- All TOML parsing invocations use the @iarna/toml parse function with file content as input
- Parsed configuration objects are validated before use in consuming components

<enforcement>
Claude Code MUST NOT skip or defer verification. All parsing operations MUST be inspected for compliance with R-TOML-001 through R-TOML-005 before code is committed.
</enforcement>