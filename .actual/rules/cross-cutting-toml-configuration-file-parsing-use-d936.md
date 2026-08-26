# Adopt @iarna/toml for TOML Configuration Parsing: Toml Configuration File Parsing Use Iarna

These rules are ALWAYS ACTIVE for all configuration file loading code, agent initialization systems, MCP server configuration parsing, and any component that reads TOML files from the filesystem for application configuration.

### Rules

- **R-TOML-001** MUST: All TOML configuration file parsing MUST use the @iarna/toml library.
- **R-TOML-002** MUST: Configuration loading MUST follow the established parse-then-validate pattern: read file content, parse with @iarna/toml, then validate the resulting object with appropriate Zod schemas.
- **R-TOML-003** MUST: Parse operations MUST be wrapped in try-catch blocks and enrich error messages with the configuration file path and guidance for users to debug TOML syntax errors.
- **R-TOML-004** SHOULD: Examine existing configuration loading implementations in the core config loader, agent initialization, and MCP propagation modules to understand and replicate the established parse-then-validate pattern.
- **R-TOML-005** MUST NOT: Use alternative TOML parsing libraries in configuration loading code paths.
- **R-TOML-006** MUST NOT: Apply TOML parsing to runtime configuration that does not originate from TOML files, configuration formats other than TOML (JSON, YAML), or TOML generation/serialization operations.

### Verify

```bash
# Discover the project's dependency manifest and verify @iarna/toml is declared
grep -r "@iarna/toml" package.json yarn.lock pnpm-lock.yaml 2>/dev/null || echo "Dependency manifest check"

# Discover and execute the project's test suite
npm test -- --testPathPattern=config 2>/dev/null || yarn test --testPathPattern=config 2>/dev/null || pnpm test --testPathPattern=config 2>/dev/null

# Search codebase for all imports of @iarna/toml
grep -r "from.*@iarna/toml" --include="*.ts" --include="*.js" src/ lib/ 2>/dev/null

# Verify no alternative TOML parsing libraries are imported
grep -r "from.*toml" --include="*.ts" --include="*.js" src/ lib/ 2>/dev/null | grep -v "@iarna/toml" || echo "No alternative TOML parsers detected"
```

**Accept when:**
- All TOML configuration parsing uses @iarna/toml library
- Configuration loading tests pass and demonstrate proper error handling for malformed TOML
- No alternative TOML parsing libraries are imported or used in configuration loading code paths
- All parse operations are wrapped in try-catch blocks with enriched error messages
- Configuration loading follows the parse-then-validate pattern with Zod schema validation

<enforcement>
Claude Code MUST NOT skip or defer verification. All TOML configuration parsing code MUST be reviewed to ensure compliance with R-TOML-001 through R-TOML-006 before accepting changes.
</enforcement>