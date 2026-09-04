# Adopt @iarna/toml and js-yaml for Structured Configuration Parsing: Yaml Parsing Operations Invoke Parse Function

These rules are ALWAYS ACTIVE for all configuration loading subsystems, agent definition loaders, subagent processors, and any component in src/core or src/agents that requires structured data parsing from TOML or YAML files.

### Rules

- **R-YAML-001** MUST: YAML parsing operations MUST invoke the parse function from js-yaml with file content as input.
- **R-YAML-002** MUST: Wrap all parsing operations in try-catch blocks and provide context-rich error messages that include file path, format type, and line number if available from parser.
- **R-YAML-003** MUST: Validate parsed configuration objects against expected schema using type guards or schema validation before passing to consuming components.
- **R-YAML-004** SHOULD: Implement a configuration loader abstraction that encapsulates format detection and parser selection, isolating direct parser dependencies to a single module.
- **R-YAML-005** MUST: Monitor security advisories for @iarna/toml and js-yaml, apply updates promptly, and validate parsed data structure before use.

### Verify

```bash
# Discover the project's dependency manifest and verify that both parsing libraries are declared as dependencies
grep -E '(@iarna/toml|js-yaml)' package.json

# Discover the project's lock artifact and confirm exact resolved versions are recorded
grep -E '(@iarna/toml|js-yaml)' package-lock.json || grep -E '(@iarna/toml|js-yaml)' yarn.lock || grep -E '(@iarna/toml|js-yaml)' pnpm-lock.yaml

# Discover and execute the project's test suite to verify parsing behavior across all configuration loading subsystems
npm test -- --testPathPattern='(config|parse|yaml|toml)'

# Verify parsing operations follow error handling patterns
grep -r 'try.*catch' src/core src/agents | grep -E '(parse|yaml|toml)'
```

**Accept when:**
- Dependency manifest declares both @iarna/toml and js-yaml as dependencies
- Lock artifact records exact resolved versions for both parsing libraries
- All test suites pass, confirming parsing operations work correctly across configuration loading subsystems
- Code inspection confirms parsing operations follow error handling and validation patterns specified in rules
- All YAML parsing invocations use js-yaml parse function with proper error handling
- Parsed configuration objects are validated against expected schema before use

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for any code that parses YAML or TOML configuration files. Violations must be caught during code review and CI pipeline checks.
</enforcement>