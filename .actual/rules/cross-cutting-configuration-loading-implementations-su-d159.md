# Adopt @iarna/toml and js-yaml for Structured Configuration Parsing: Configuration Loading Implementations Support Multiple Format

These rules are ALWAYS ACTIVE for all configuration loading implementations, agent definition loaders, subagent processors, and any component in src/core or src/agents that requires structured data parsing from files in TOML, YAML, or JSON formats.

### Rules

- **R-CONFIG-001** SHOULD: Configuration loading implementations SHOULD support multiple format detection and route to the appropriate parser based on file extension or content inspection.
- **R-CONFIG-002** MUST: Wrap parsing operations in try-catch blocks and provide context-rich error messages that include file path, format type, and line number if available from parser.
- **R-CONFIG-003** SHOULD: Implement a configuration loader abstraction that encapsulates format detection and parser selection, isolating direct parser dependencies to a single module.
- **R-CONFIG-004** MUST: Validate parsed configuration objects against expected schema using type guards or schema validation before passing to consuming components.
- **R-CONFIG-005** MUST: Use @iarna/toml and js-yaml as the designated parsing libraries for TOML and YAML configuration files respectively.

### Verify

```bash
# Discover the project's dependency manifest and verify that both parsing libraries are declared as dependencies
grep -E '(@iarna/toml|js-yaml)' package.json

# Discover the project's lock artifact and confirm exact resolved versions are recorded
grep -E '(@iarna/toml|js-yaml)' package-lock.json || grep -E '(@iarna/toml|js-yaml)' yarn.lock || grep -E '(@iarna/toml|js-yaml)' pnpm-lock.yaml

# Discover and execute the project's test suite to verify parsing behavior across all configuration loading subsystems
npm test -- --testPathPattern='(config|loader|parser)'
```

**Accept when:**
- Dependency manifest declares both @iarna/toml and js-yaml as dependencies
- Lock artifact records exact resolved versions for both parsing libraries
- All test suites pass, confirming parsing operations work correctly across configuration loading subsystems
- Code inspection confirms parsing operations follow error handling and validation patterns specified in rules
- Configuration loading implementations include format detection logic routing to appropriate parser
- All parse operations are wrapped in try-catch with context-rich error messages
- Parsed configuration is validated against schema before use

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration loading implementations MUST comply with these rules before merge.
</enforcement>