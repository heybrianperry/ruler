# Adopt @iarna/toml for TOML Configuration Parsing: Configuration Loading Functions Separate File Reading

These rules are ALWAYS ACTIVE for all configuration file loading code paths, including core config loaders, agent initialization systems, MCP server configuration parsing, and any component that reads TOML files from the filesystem for application configuration.

### Rules

- **R-TOML-001** MUST: Use @iarna/toml library for all TOML parsing operations in configuration loading code.
- **R-TOML-002** SHOULD: Separate file reading, TOML parsing, and validation into distinct steps for clarity and error handling.
- **R-TOML-003** SHOULD: Wrap parse operations in try-catch blocks and enrich error messages with the configuration file path and guidance for users to debug TOML syntax errors.
- **R-TOML-004** MUST: Validate parsed TOML objects using appropriate Zod schemas before consuming configuration values.
- **R-TOML-005** MUST NOT: Introduce alternative TOML parsing libraries without documented technical justification and engineering team approval.

### Verify

```bash
# Discover the project's dependency manifest and verify @iarna/toml is declared
grep -r "@iarna/toml" package.json yarn.lock pnpm-lock.yaml 2>/dev/null | head -5

# Search the codebase for all imports of @iarna/toml
grep -r "from.*@iarna/toml" --include="*.ts" --include="*.js" src/ 2>/dev/null

# Verify no alternative TOML parsing libraries are imported
grep -r "toml" --include="*.ts" --include="*.js" src/ | grep -v "@iarna/toml" | grep -v "TOML" | grep -v "toml-" || echo "No alternative TOML parsers detected"

# Execute the project's test suite to verify configuration loading tests pass
npm test -- --testPathPattern=config 2>/dev/null || yarn test --testPathPattern=config 2>/dev/null || pnpm test --testPathPattern=config 2>/dev/null
```

**Accept when:**
- All TOML configuration parsing uses @iarna/toml library exclusively
- Configuration loading tests pass and demonstrate proper error handling for malformed TOML
- No alternative TOML parsing libraries are imported or used in configuration loading code paths
- Parse operations include try-catch blocks with enriched error messages containing file paths
- Parsed TOML objects are validated with Zod schemas before use

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration loading code must be audited to confirm @iarna/toml usage and proper separation of file reading, parsing, and validation steps.
</enforcement>