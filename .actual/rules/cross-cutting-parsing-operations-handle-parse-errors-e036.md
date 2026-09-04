# Adopt @iarna/toml and js-yaml for Structured Configuration Parsing: Parsing Operations Handle Parse Errors Provide

These rules are ALWAYS ACTIVE for all configuration loading subsystems, agent definition loaders, subagent processors, and any component in src/core or src/agents that directly parses structured configuration files from TOML, YAML, or JSON sources.

### Rules

- **R-PARSE-001** MUST: Parsing operations MUST handle parse errors and provide meaningful error messages indicating which file and format failed to parse.
- **R-PARSE-002** MUST: Wrap parsing operations in try-catch blocks and provide context-rich error messages that include file path, format type, and line number if available from parser.
- **R-PARSE-003** MUST: Validate parsed configuration objects against expected schema using type guards or schema validation before passing to consuming components.
- **R-PARSE-004** SHOULD: Implement a configuration loader abstraction that encapsulates format detection and parser selection, isolating direct parser dependencies to a single module.
- **R-PARSE-005** MUST: Use @iarna/toml and js-yaml as the designated parsing libraries for TOML and YAML configuration files respectively.

### Verify

```bash
# Discover the project's dependency manifest and verify that both parsing libraries are declared as dependencies
grep -E '(@iarna/toml|js-yaml)' package.json || echo "ERROR: Parsing libraries not found in package.json"

# Discover the project's lock artifact and confirm exact resolved versions are recorded
grep -E '(@iarna/toml|js-yaml)' package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null | head -20 || echo "ERROR: Lock artifact not found"

# Discover and execute the project's test suite to verify parsing behavior across all configuration loading subsystems
npm test -- --testPathPattern='(config|parse|loader)' 2>&1 | tail -20
```

**Accept when:**
- Dependency manifest declares both @iarna/toml and js-yaml as dependencies
- Lock artifact records exact resolved versions for both parsing libraries
- All test suites pass, confirming parsing operations work correctly across configuration loading subsystems
- Code inspection confirms parsing operations follow error handling and validation patterns specified in these rules
- All parsing operations include try-catch blocks with context-rich error messages containing file path, format type, and line number information
- Parsed configuration objects are validated against expected schema before use

<enforcement>
Claude Code MUST NOT skip or defer verification. All parsing operations MUST include proper error handling with meaningful error messages. Code review MUST verify compliance with R-PARSE-001 through R-PARSE-005 before merge.
</enforcement>