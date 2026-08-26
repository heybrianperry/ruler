# Adopt @iarna/toml for TOML Configuration Parsing: Parse Errors Iarna Toml Caught Wrapped

These rules are ALWAYS ACTIVE for all configuration file loading code paths, including core config loaders, agent initialization systems, MCP propagation logic, and any component that reads TOML files from the filesystem for application configuration.

### Rules

- **R-TOML-001** MUST: Use @iarna/toml library for all TOML parsing operations in configuration loading.
- **R-TOML-002** SHOULD: Catch parse errors from @iarna/toml and wrap them with context about which configuration file failed to parse.
- **R-TOML-003** SHOULD: Follow the established parse-then-validate pattern: read file content, parse with @iarna/toml, then validate the resulting object with appropriate Zod schemas.
- **R-TOML-004** SHOULD: Wrap parse operations in try-catch blocks and enrich error messages with the configuration file path and guidance for users to debug TOML syntax errors.
- **R-TOML-005** MUST: Not introduce alternative TOML parsing libraries in configuration loading code paths.

### Verify

```bash
# Discover the project's dependency manifest and verify @iarna/toml is declared
grep -r "@iarna/toml" package.json package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null || echo "Dependency manifest check"

# Search the codebase for all imports of @iarna/toml
grep -r "from.*@iarna/toml" --include="*.ts" --include="*.js" src/ lib/ 2>/dev/null || echo "No @iarna/toml imports found"

# Verify no alternative TOML parsing libraries are imported
grep -r "from.*toml" --include="*.ts" --include="*.js" src/ lib/ | grep -v "@iarna/toml" || echo "No alternative TOML parsers detected"

# Discover and execute the project's test suite
npm test 2>/dev/null || yarn test 2>/dev/null || pnpm test 2>/dev/null || echo "Test suite execution"
```

**Accept when:**
- All TOML configuration parsing uses @iarna/toml library
- Configuration loading tests pass and demonstrate proper error handling for malformed TOML
- No alternative TOML parsing libraries are imported or used in configuration loading code paths
- Parse errors are caught and wrapped with file path context and user-friendly guidance
- The parse-then-validate pattern is consistently applied across all configuration loading components

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration file loading code MUST comply with these rules before acceptance.
</enforcement>