# Adopt @iarna/toml for TOML Configuration Parsing: Before Implementing Any Code That Uses

These rules are ALWAYS ACTIVE for all code that loads, parses, or processes TOML configuration files in core infrastructure components, agent initialization systems, and MCP server management.

### Rules

- **R-TOML-001** MUST: Before implementing any code that uses @iarna/toml, discover the project's dependency lock artifact, resolve the exact installed version, and verify API compatibility against that version's official documentation.
- **R-TOML-002** MUST: All TOML configuration parsing in core infrastructure components (ConfigLoader, agent initialization, MCP propagation) use @iarna/toml library exclusively.
- **R-TOML-003** MUST: Follow the established parse-then-validate pattern: read file content, parse with @iarna/toml, then validate the resulting object with appropriate Zod schemas.
- **R-TOML-004** MUST: Wrap parse operations in try-catch blocks and enrich error messages with the configuration file path and guidance for users to debug TOML syntax errors.
- **R-TOML-005** SHOULD: Examine existing configuration loading implementations in core config loader, agent initialization, and MCP propagation modules to understand the established pattern before adding new configuration file loading.
- **R-TOML-006** MAY: Monitor dependency security advisories and maintain awareness of alternative TOML parsing libraries as mitigation against library maintenance risks.

### Verify

```bash
# 1. Discover the project's dependency manifest and verify @iarna/toml is declared
grep -r "@iarna/toml" package.json yarn.lock pnpm-lock.yaml 2>/dev/null || echo "Dependency manifest check"

# 2. Identify the exact resolved version from the lock artifact
# (Adjust for project's package manager: npm, yarn, or pnpm)
cat package-lock.json | grep -A 2 '"@iarna/toml"' || \
cat yarn.lock | grep -A 5 '@iarna/toml' || \
cat pnpm-lock.yaml | grep -A 5 '@iarna/toml'

# 3. Search codebase for all imports of @iarna/toml
grep -r "from.*@iarna/toml" --include="*.ts" --include="*.js" src/ lib/ || echo "No imports found"

# 4. Verify all TOML parsing follows parse-then-validate pattern
grep -r "@iarna/toml" --include="*.ts" --include="*.js" -A 3 src/ | grep -E "(parse|validate)" || echo "Pattern verification"

# 5. Search for alternative TOML parsing libraries
grep -r "toml" --include="*.ts" --include="*.js" src/ | grep -v "@iarna/toml" | grep -E "(import|require)" || echo "No alternative parsers detected"

# 6. Run configuration loading tests
npm test -- --testPathPattern="config|toml" || \
yarn test --testPathPattern="config|toml" || \
pnpm test --testPathPattern="config|toml"
```

**Accept when:**
- All TOML configuration parsing uses @iarna/toml library exclusively
- Configuration loading tests pass and demonstrate proper error handling for malformed TOML
- No alternative TOML parsing libraries are imported or used in configuration loading code paths
- Error handling wraps parse operations with file path context and user-friendly guidance
- The exact installed version of @iarna/toml has been verified against official documentation before use

<enforcement>
Claude Code MUST NOT skip or defer verification. Before writing any code that uses @iarna/toml, execute the verify commands to confirm the exact version and validate API compatibility. Code review MUST verify that new configuration loading code uses @iarna/toml and follows the parse-then-validate pattern.
</enforcement>