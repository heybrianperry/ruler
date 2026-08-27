# Adopt @iarna/toml for TOML Configuration Parsing: Toml Configuration File Parsing Use Adopted

These rules are ALWAYS ACTIVE for all modules that load TOML configuration files from the filesystem, including agent initialization and configuration loading logic, MCP server configuration processing, application-level configuration management, and subagent processor configuration parsing.

### Rules

- **R-TOML-001** MUST: All TOML configuration file parsing MUST use the adopted TOML parsing library's API functions.
- **R-TOML-002** MUST: TOML parsing operations MUST include error handling that distinguishes between file system errors (file not found, permission denied) and TOML parsing errors (invalid syntax, type mismatches).
- **R-TOML-003** MUST: Parsed TOML configuration objects MUST be validated against a schema before consuming configuration values.
- **R-TOML-004** SHOULD: Follow the standard parsing pattern: read file content as string using filesystem APIs, then pass the string to the TOML library's parse function, then validate the resulting object against a schema.
- **R-TOML-005** MUST: No alternative TOML parsing libraries MAY be introduced into the dependency manifest.

### Verify

```bash
# Discover the project's dependency manifest and identify the TOML parsing library entry
grep -E '(toml|@iarna)' package.json package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null | head -20

# Locate and inspect the dependency lock artifact to confirm the TOML library is installed at a specific resolved version
if [ -f package-lock.json ]; then jq '.dependencies | keys[] | select(contains("toml"))' package-lock.json; fi
if [ -f yarn.lock ]; then grep -A 2 'toml' yarn.lock | head -20; fi
if [ -f pnpm-lock.yaml ]; then grep -A 2 'toml' pnpm-lock.yaml | head -20; fi

# Search the codebase for imports of the TOML parsing library and verify all usage follows the standard parsing pattern
find . -type f \( -name '*.ts' -o -name '*.js' \) ! -path './node_modules/*' -exec grep -l 'import.*toml\|require.*toml' {} \;

# Verify no alternative TOML parsing libraries are present
grep -E '(yaml|js-yaml|json5|hjson)' package.json 2>/dev/null || echo 'No conflicting parsers detected'

# Execute the project's test suite and verify TOML parsing tests pass
npm test -- --testPathPattern=toml 2>/dev/null || yarn test --testPathPattern=toml 2>/dev/null || echo 'Run project test suite manually'
```

**Accept when:**
- All TOML configuration parsing uses the adopted library's API functions
- No alternative TOML parsing libraries are present in the dependency manifest
- TOML parsing operations include error handling for invalid syntax and file system errors
- Test coverage exists for TOML configuration parsing across affected modules
- Parsed configuration is validated against schemas before use
- Code review confirms standard parsing pattern is followed

<enforcement>
Claude Code MUST NOT skip or defer verification. All TOML parsing code MUST be reviewed against R-TOML-001 through R-TOML-005 before acceptance. Pull requests introducing alternative TOML libraries or non-standard parsing patterns MUST be rejected with reference to this rule file.
</enforcement>