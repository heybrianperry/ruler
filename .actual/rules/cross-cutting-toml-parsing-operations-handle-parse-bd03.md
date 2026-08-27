# Adopt @iarna/toml for TOML Configuration Parsing: Toml Parsing Operations Handle Parse Errors

These rules are ALWAYS ACTIVE for all modules that load TOML configuration files from the filesystem, including agent initialization and configuration loading logic, MCP server configuration processing, application-level configuration management, and subagent processor configuration parsing.

### Rules

- **R-TOML-001** MUST: TOML parsing operations MUST handle parse errors and provide meaningful error messages when configuration files contain invalid TOML syntax.
- **R-TOML-002** MUST: All TOML configuration parsing MUST use the adopted library's API functions consistently across all modules.
- **R-TOML-003** MUST: Error handling MUST distinguish between file system errors (file not found, permission denied) and TOML parsing errors (invalid syntax, type mismatches) to provide actionable error messages.
- **R-TOML-004** MUST: Parsed TOML content MUST be validated against a schema before consuming configuration values.
- **R-TOML-005** SHOULD: Follow the standard parsing pattern: read file content as string using filesystem APIs, then pass the string to the TOML library's parse function, then validate the resulting object against a schema.

### Verify

```bash
# Discover the project's dependency manifest and identify the TOML parsing library entry
grep -r "@iarna/toml" package.json package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null || echo "TOML library not found in manifest"

# Locate and inspect the dependency lock artifact to confirm the TOML library is installed at a specific resolved version
if [ -f package-lock.json ]; then
  jq '.dependencies["@iarna/toml"] // .packages["node_modules/@iarna/toml"]' package-lock.json
elif [ -f yarn.lock ]; then
  grep -A 5 '"@iarna/toml"' yarn.lock
elif [ -f pnpm-lock.yaml ]; then
  grep -A 5 '@iarna/toml' pnpm-lock.yaml
fi

# Search the codebase for imports of the TOML parsing library and verify all usage follows the standard parsing pattern
grep -r "import.*@iarna/toml" --include="*.ts" --include="*.js" src/ || echo "No TOML imports found"
grep -r "require.*@iarna/toml" --include="*.ts" --include="*.js" src/ || echo "No TOML requires found"

# Verify no alternative TOML parsing libraries are present
grep -E "toml|yaml|json5" package.json | grep -v "@iarna/toml" || echo "No alternative TOML libraries detected"

# Execute the project's test suite and verify TOML parsing tests pass
npm test -- --testPathPattern="toml|config" 2>/dev/null || echo "Run test suite to verify TOML parsing"
```

**Accept when:**
- All TOML configuration parsing uses the adopted library's API functions
- No alternative TOML parsing libraries are present in the dependency manifest
- TOML parsing operations include error handling for invalid syntax
- Test coverage exists for TOML configuration parsing across affected modules
- File system errors are distinguished from TOML parsing errors in error messages
- Parsed configuration is validated against schemas before consumption

<enforcement>
Claude Code MUST NOT skip or defer verification. All TOML parsing code MUST be reviewed to ensure it follows R-TOML-001 through R-TOML-005. Pull requests introducing alternative TOML libraries or non-standard parsing patterns MUST be rejected with reference to this rule file.
</enforcement>