# Adopt @iarna/toml and js-yaml for Structured Configuration Parsing: Configuration Parsing Subsystems Use Iarna Toml

These rules are ALWAYS ACTIVE for all configuration parsing subsystems, agent definition loaders, subagent processors, and any component in src/core or src/agents that requires structured data parsing from TOML or YAML files.

### Rules

- **R-CONFIG-PARSE-001** MUST: Configuration parsing subsystems MUST use @iarna/toml for TOML format parsing and js-yaml for YAML format parsing.
- **R-CONFIG-PARSE-002** MUST: Wrap all parsing operations in try-catch blocks and provide context-rich error messages that include file path, format type, and line number if available from parser.
- **R-CONFIG-PARSE-003** MUST: Validate parsed configuration objects against expected schema using type guards or schema validation before passing to consuming components.
- **R-CONFIG-PARSE-004** SHOULD: Implement a configuration loader abstraction that encapsulates format detection and parser selection, isolating direct parser dependencies to a single module.
- **R-CONFIG-PARSE-005** MUST: Confirm every API, class, or function used from parsing libraries exists in the exact resolved version's documentation before implementation.

### Verify

```bash
# Discover the project's dependency manifest and verify both parsing libraries are declared
grep -E '@iarna/toml|js-yaml' package.json

# Discover the project's lock artifact and confirm exact resolved versions
grep -A 2 -E '@iarna/toml|js-yaml' package-lock.json || grep -A 2 -E '@iarna/toml|js-yaml' yarn.lock

# Execute the project's test suite to verify parsing behavior
npm test -- --testPathPattern='config|parse'

# Verify parsing operations follow error handling patterns
grep -r 'try.*catch' src/core src/agents | grep -E 'toml|yaml'
```

**Accept when:**
- Dependency manifest declares both @iarna/toml and js-yaml as dependencies
- Lock artifact records exact resolved versions for both parsing libraries
- All test suites pass, confirming parsing operations work correctly across configuration loading subsystems
- Code inspection confirms parsing operations follow error handling and validation patterns specified in rules
- All parsing calls include try-catch blocks with context-rich error messages
- Parsed configuration objects are validated against expected schema before use

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing code MUST comply with R-CONFIG-PARSE-001 through R-CONFIG-PARSE-005 before merge. CI pipeline MUST fail if parsing library dependencies are missing or have security vulnerabilities. Code review MUST block merge if parsing operations lack proper error handling.
</enforcement>