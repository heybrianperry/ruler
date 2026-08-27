# Adopt @iarna/toml for TOML Configuration Parsing: Toml Parsing Occur Immediately After File

These rules are ALWAYS ACTIVE for all modules that load TOML configuration files from the filesystem, including agent initialization and configuration loading logic, MCP server configuration processing, application-level configuration management, and subagent processor configuration parsing.

### Rules

- **R-TOML-001** SHOULD: TOML parsing SHOULD occur immediately after file content is read and before any business logic consumes the configuration data.
- **R-TOML-002** MUST: All TOML configuration parsing uses the adopted library's API functions.
- **R-TOML-003** MUST: Error handling distinguishes between file system errors (file not found, permission denied) and TOML parsing errors (invalid syntax, type mismatches).
- **R-TOML-004** MUST: Parsed configuration objects are validated against a schema before consuming configuration values.
- **R-TOML-005** SHOULD: When adding new configuration files, ensure TOML syntax is validated during development and that parsing errors are caught in CI before deployment.

### Verify

```bash
# Discover the project's dependency manifest and identify the TOML parsing library entry
grep -r "@iarna/toml" package.json package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null || echo "TOML library not found in manifest"

# Locate and inspect the dependency lock artifact to confirm the TOML library is installed at a specific resolved version
if [ -f package-lock.json ]; then
  jq '.dependencies["@iarna/toml"].version' package-lock.json
elif [ -f yarn.lock ]; then
  grep -A 2 '"@iarna/toml"' yarn.lock | head -3
elif [ -f pnpm-lock.yaml ]; then
  grep -A 2 '@iarna/toml' pnpm-lock.yaml | head -3
fi

# Search the codebase for imports of the TOML parsing library and verify all usage follows the standard parsing pattern
grep -r "from.*@iarna/toml\|require.*@iarna/toml" --include="*.ts" --include="*.js" . 2>/dev/null | wc -l

# Verify no alternative TOML parsing libraries are present in the dependency manifest
grep -E "toml|yaml" package.json | grep -v "@iarna/toml" || echo "No alternative TOML libraries detected"

# Execute the project's test suite and verify TOML parsing tests pass
npm test -- --testPathPattern="toml|config" 2>/dev/null || echo "Run test suite to verify TOML parsing"
```

**Accept when:**
- All TOML configuration parsing uses the adopted library's API functions
- No alternative TOML parsing libraries are present in the dependency manifest
- TOML parsing operations include error handling for invalid syntax
- Test coverage exists for TOML configuration parsing across affected modules
- Parsed configuration is validated against schemas before consumption
- File system errors are distinguished from TOML parsing errors in error handling

<enforcement>
Claude Code MUST NOT skip or defer verification. All TOML parsing code must be reviewed against these rules before acceptance. Dependency audits must confirm no alternative TOML libraries are introduced. Static analysis must verify imports match the adopted library.
</enforcement>