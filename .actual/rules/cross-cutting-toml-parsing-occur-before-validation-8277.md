# Adopt @iarna/toml for TOML Configuration Parsing: Toml Parsing Occur Before Validation Logic

These rules are ALWAYS ACTIVE for all configuration file loading code paths, agent initialization systems, MCP server configuration parsing, and any component that reads TOML files from the filesystem for application configuration.

### Rules

- **R-TOML-001** MUST: TOML parsing MUST occur before validation logic is applied to configuration objects.
- **R-TOML-002** MUST: All TOML configuration parsing MUST use the @iarna/toml library.
- **R-TOML-003** MUST: Configuration loading code MUST follow the established parse-then-validate pattern: read file content, parse with @iarna/toml, then validate the resulting object with appropriate Zod schemas.
- **R-TOML-004** MUST: Parse operations MUST be wrapped in try-catch blocks and enrich error messages with the configuration file path and guidance for users to debug TOML syntax errors.
- **R-TOML-005** SHOULD: When adding new configuration file loading, examine existing implementations in core config loader, agent initialization, and MCP propagation modules to understand and replicate the established pattern.

### Verify

```bash
# Discover the project's dependency manifest and verify @iarna/toml is declared
grep -r "@iarna/toml" package.json package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null || echo "Dependency manifest check"

# Search the codebase for all imports of @iarna/toml
grep -r "from.*@iarna/toml" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" .

# Verify no alternative TOML parsing libraries are imported
grep -r "from.*toml" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" . | grep -v "@iarna/toml" || echo "No alternative TOML parsers detected"

# Execute the project's test suite to verify configuration loading tests pass
npm test -- --testPathPattern=config 2>/dev/null || yarn test --testPathPattern=config 2>/dev/null || echo "Run project test suite"
```

**Accept when:**
- All TOML configuration parsing uses @iarna/toml library
- Configuration loading tests pass and demonstrate proper error handling for malformed TOML
- No alternative TOML parsing libraries are imported or used in configuration loading code paths
- Parse operations are wrapped in try-catch blocks with enriched error messages
- New configuration loading code follows the parse-then-validate pattern

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration file loading code MUST comply with R-TOML-001 through R-TOML-005 before submission.
</enforcement>