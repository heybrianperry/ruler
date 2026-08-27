# Adopt @iarna/toml for TOML Configuration Parsing: Modules That Parse Toml Configuration Files

These rules are ALWAYS ACTIVE for all modules that load TOML configuration files from the filesystem, including agent initialization and configuration loading logic, MCP server configuration processing, application-level configuration management, and subagent processor configuration parsing.

### Rules

- **R-TOML-001** SHOULD: Modules that parse TOML configuration files SHOULD validate the parsed output against a schema before consuming configuration values.
- **R-TOML-002** MUST: All TOML configuration parsing MUST use the adopted library's API functions, not alternative TOML parsing libraries.
- **R-TOML-003** MUST: TOML parsing operations MUST include error handling that distinguishes between file system errors (file not found, permission denied) and TOML parsing errors (invalid syntax, type mismatches).
- **R-TOML-004** SHOULD: When adding new configuration files, SHOULD ensure TOML syntax is validated during development and that parsing errors are caught in CI before deployment.
- **R-TOML-005** MUST: Standard parsing pattern MUST follow: read file content as string using filesystem APIs, then pass the string to the TOML library's parse function, then validate the resulting object against a schema before consuming configuration values.

### Verify

```bash
# Discover the project's dependency manifest and identify the TOML parsing library entry
grep -r "toml" package.json yarn.lock pnpm-lock.yaml 2>/dev/null | head -20

# Locate and inspect the dependency lock artifact to confirm the TOML library is installed at a specific resolved version
if [ -f "package-lock.json" ]; then jq '.dependencies | keys[] | select(contains("toml"))' package-lock.json; fi
if [ -f "yarn.lock" ]; then grep -A 5 "^@iarna/toml" yarn.lock; fi
if [ -f "pnpm-lock.yaml" ]; then grep -A 5 "@iarna/toml" pnpm-lock.yaml; fi

# Search the codebase for imports of the TOML parsing library and verify all usage follows the standard parsing pattern
grep -r "from.*toml\|require.*toml" --include="*.ts" --include="*.js" src/ lib/ 2>/dev/null

# Verify no alternative TOML parsing libraries are imported
grep -r "from.*toml-js\|from.*toml-parse\|require.*toml-js\|require.*toml-parse" --include="*.ts" --include="*.js" src/ lib/ 2>/dev/null || echo "No alternative TOML libraries detected"

# Execute the project's test suite and verify TOML parsing tests pass
npm test -- --testPathPattern="toml|config" 2>/dev/null || yarn test --testPathPattern="toml|config" 2>/dev/null || pnpm test --testPathPattern="toml|config" 2>/dev/null
```

**Accept when:**
- All TOML configuration parsing uses the adopted library's API functions
- No alternative TOML parsing libraries are present in the dependency manifest
- TOML parsing operations include error handling for invalid syntax and file system errors
- Test coverage exists for TOML configuration parsing across affected modules
- Parsed TOML output is validated against a schema before consumption
- Standard parsing pattern (read → parse → validate) is followed consistently

<enforcement>
Claude Code MUST NOT skip or defer verification. All TOML parsing code MUST be reviewed against these rules before acceptance. Pull requests introducing alternative TOML libraries or non-standard parsing patterns MUST be rejected with reference to this ADR.
</enforcement>