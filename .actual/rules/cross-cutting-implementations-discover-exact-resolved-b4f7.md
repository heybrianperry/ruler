# Adopt @iarna/toml for TOML Configuration Parsing: Implementations Discover Exact Resolved Version Iarna

These rules are ALWAYS ACTIVE for all configuration file loading where the source format is TOML, including agent configuration initialization, system configuration loading, MCP server definition parsing, and any subsystem that reads TOML-formatted files from the filesystem.

### Rules

- **R-TOML-001** MUST: Implementations MUST discover the exact resolved version of @iarna/toml from the project's dependency lock file before writing code that uses the library.
- **R-TOML-002** MUST: All TOML parsing import statements MUST reference @iarna/toml exclusively; no alternative TOML parsing libraries are permitted without architectural review and exception approval.
- **R-TOML-003** MUST: Isolate TOML parsing behind clear interfaces or utility functions to enable future library substitution without widespread code changes.
- **R-TOML-004** MUST: Implement consistent error handling for TOML parsing failures that includes file path context and actionable error messages for configuration authors.
- **R-TOML-005** SHOULD: Follow the established pattern of parse-then-validate: use @iarna/toml for parsing, then apply schema validation separately.

### Verify

```bash
# 1. Discover the project's dependency manifest and confirm @iarna/toml is declared
grep -r "@iarna/toml" package.json package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null | head -5

# 2. Inspect the lock file to determine exact resolved version
if [ -f package-lock.json ]; then
  jq '.dependencies["@iarna/toml"].version' package-lock.json
elif [ -f yarn.lock ]; then
  grep -A 2 '"@iarna/toml@' yarn.lock | grep version
elif [ -f pnpm-lock.yaml ]; then
  grep -A 2 '@iarna/toml' pnpm-lock.yaml | grep version
fi

# 3. Search codebase for TOML parsing imports and verify all use @iarna/toml
grep -r "from.*toml" --include="*.ts" --include="*.js" . | grep -v node_modules | grep -v ".actual"

# 4. Verify no alternative TOML parsing libraries are present
grep -E "(toml|@ltd/j-toml)" package.json | grep -v "@iarna/toml"

# 5. Locate and execute configuration loading tests
npm test -- --testPathPattern="config|toml" 2>/dev/null || yarn test --testPathPattern="config|toml" 2>/dev/null || pnpm test --testPathPattern="config|toml" 2>/dev/null
```

**Accept when:**
- Dependency manifest declares @iarna/toml and no alternative TOML parsing libraries are present
- Lock file contains exact resolved version of @iarna/toml
- All TOML parsing import statements reference @iarna/toml exclusively
- No alternative TOML parsers (toml, @ltd/j-toml, etc.) are found in package.json
- Configuration loading tests pass, demonstrating successful TOML parsing across all configuration contexts
- TOML parsing is isolated behind utility functions or clear interfaces

<enforcement>
Claude Code MUST NOT skip or defer verification. Before writing code that uses @iarna/toml, execute the verify commands in order to confirm the exact resolved version and validate that all existing TOML parsing uses @iarna/toml exclusively. Reject code that introduces alternative TOML parsing libraries or bypasses the standard parsing approach without documented exception approval.
</enforcement>