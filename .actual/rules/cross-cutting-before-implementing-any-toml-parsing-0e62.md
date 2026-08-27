# Adopt @iarna/toml for TOML Configuration Parsing: Before Implementing Any Toml Parsing Logic

These rules are ALWAYS ACTIVE for all modules that load TOML configuration files from the filesystem, including agent initialization and configuration loading logic, MCP server configuration processing, application-level configuration management, and subagent processor configuration parsing.

### Rules

- **R-TOML-001** MUST: Before implementing any TOML parsing logic, discover the project's dependency lock artifact and resolve the exact installed version of the TOML parsing library.
- **R-TOML-002** MUST: Use only the adopted TOML parsing library's API functions for all TOML configuration parsing operations across the codebase.
- **R-TOML-003** MUST: Follow the standard parsing pattern: read file content as string using filesystem APIs, then pass the string to the TOML library's parse function, then validate the resulting object against a schema before consuming configuration values.
- **R-TOML-004** MUST: Distinguish between file system errors (file not found, permission denied) and TOML parsing errors (invalid syntax, type mismatches) in error handling to provide actionable error messages.
- **R-TOML-005** MUST: Validate TOML syntax during development and ensure parsing errors are caught in CI before deployment when adding new configuration files.
- **R-TOML-006** SHOULD: Maintain test coverage for TOML parsing to enable library migration if needed and to validate parsed configuration against schemas.

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
- The exact resolved version of the TOML library is documented and verified from the lock artifact
- All imports of the TOML library match the adopted library name

<enforcement>
Clause Code MUST NOT skip or defer verification. All TOML parsing implementations MUST be reviewed against R-TOML-001 through R-TOML-006. Pull requests introducing alternative TOML libraries are rejected with reference to this rule file. Code using non-standard TOML parsing patterns is flagged in review and must be refactored. CI MUST fail if dependency manifest contains multiple TOML parsing libraries.
</enforcement>