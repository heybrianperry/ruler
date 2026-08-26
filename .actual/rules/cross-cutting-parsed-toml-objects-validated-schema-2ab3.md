# Adopt @iarna/toml for TOML Configuration Parsing: Parsed Toml Objects Validated Schema Validation

These rules are ALWAYS ACTIVE for all configuration file loading code, agent initialization systems, MCP server configuration parsing, and any component that reads TOML files from the filesystem for application configuration.

### Rules

- **R-TOML-001** MUST: Use @iarna/toml library for all TOML parsing operations in configuration loading code paths.
- **R-TOML-002** MUST: Validate parsed TOML objects using schema validation libraries (such as Zod) before consuming them in application logic.
- **R-TOML-003** MUST: Wrap TOML parse operations in try-catch blocks and enrich error messages with the configuration file path and TOML syntax debugging guidance.
- **R-TOML-004** SHOULD: Follow the established parse-then-validate pattern when adding new configuration file loading code.
- **R-TOML-005** MAY: Parsed TOML objects MAY be validated using schema validation libraries before being consumed by application logic.

### Verify

```bash
# Discover the project's dependency manifest and verify @iarna/toml is declared
grep -r "@iarna/toml" package.json yarn.lock pnpm-lock.yaml 2>/dev/null || echo "Dependency manifest check"

# Discover and execute the project's test suite
npm test -- --testPathPattern=config 2>/dev/null || yarn test --testPathPattern=config 2>/dev/null || pnpm test --testPathPattern=config 2>/dev/null

# Search for all imports of @iarna/toml and verify parse-then-validate pattern
grep -r "from.*@iarna/toml" --include="*.ts" --include="*.js" src/ lib/ 2>/dev/null
grep -r "require.*@iarna/toml" --include="*.ts" --include="*.js" src/ lib/ 2>/dev/null

# Verify no alternative TOML parsing libraries are used
grep -r "toml-js\|toml-parse\|@ltd/j-toml" --include="*.ts" --include="*.js" src/ lib/ 2>/dev/null && echo "WARNING: Alternative TOML parser detected" || echo "No alternative TOML parsers found"
```

**Accept when:**
- All TOML configuration parsing uses @iarna/toml library
- Configuration loading tests pass and demonstrate proper error handling for malformed TOML
- No alternative TOML parsing libraries are imported or used in configuration loading code paths
- Parse operations include try-catch blocks with enriched error messages
- Parsed TOML objects are validated against schemas before use

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration loading code must be audited to confirm @iarna/toml usage and proper error handling before acceptance.
</enforcement>