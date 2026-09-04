# Adopt @iarna/toml and js-yaml for Structured Configuration Parsing: Before Implementing Any Code That Invokes

These rules are ALWAYS ACTIVE for all code that implements configuration loading subsystems, agent definition loaders, subagent processors, and any component in src/core or src/agents that directly parses structured configuration files from TOML, YAML, or JSON sources.

### Rules

- **R-TOML-YAML-001** MUST: Before implementing any code that invokes parsing library APIs, discover the project's dependency lock artifact, resolve the exact installed version of each parsing library (@iarna/toml and js-yaml), and verify API compatibility against that version's official documentation.
- **R-TOML-YAML-002** MUST: Wrap all parsing operations in try-catch blocks and provide context-rich error messages that include file path, format type, and line number if available from the parser.
- **R-TOML-YAML-003** MUST: Validate parsed configuration objects against expected schema using type guards or schema validation before passing to consuming components.
- **R-TOML-YAML-004** SHOULD: Implement a configuration loader abstraction that encapsulates format detection and parser selection, isolating direct parser dependencies to a single module.
- **R-TOML-YAML-005** MUST: Confirm every API, class, or function to be called exists in the exact resolved version's official documentation before using it; do not rely on training-data recall.

### Verify

```bash
# 1. Discover the project's dependency manifest and verify both parsing libraries are declared
grep -E '@iarna/toml|js-yaml' package.json

# 2. Discover the project's lock artifact and confirm exact resolved versions
grep -A 2 '@iarna/toml\|js-yaml' package-lock.json || grep -A 2 '@iarna/toml\|js-yaml' yarn.lock || grep -A 2 '@iarna/toml\|js-yaml' pnpm-lock.yaml

# 3. Discover and execute the project's test suite to verify parsing behavior
npm test -- --testPathPattern='config|parser|loader'

# 4. Verify parsing operations follow error handling patterns
grep -r 'try.*catch' src/core src/agents | grep -E 'toml|yaml|parse'

# 5. Verify validation is applied after parsing
grep -r 'validate\|schema' src/core src/agents | grep -E 'toml|yaml|parse'
```

**Accept when:**
- Dependency manifest declares both @iarna/toml and js-yaml as dependencies
- Lock artifact records exact resolved versions for both parsing libraries
- All test suites pass, confirming parsing operations work correctly across configuration loading subsystems
- Code inspection confirms all parsing operations are wrapped in try-catch blocks with context-rich error messages
- Code inspection confirms parsed configuration objects are validated against expected schema before use
- All parsing API calls are verified against the exact resolved version's official documentation

<enforcement>
Claude Code MUST NOT skip or defer verification. Before writing any code that invokes @iarna/toml or js-yaml APIs, execute the verify commands above to confirm dependency versions and validate all API usage against official documentation for those exact versions. Code review MUST block merge if parsing operations lack proper error handling or validation.
</enforcement>