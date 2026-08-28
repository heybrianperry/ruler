# Adopt @iarna/toml for TOML Configuration Parsing: Parsing Errors Iarna Toml Caught Wrapped

These rules are ALWAYS ACTIVE for all configuration file loading code where the source format is TOML, including agent configuration initialization, system configuration loading, MCP server definition parsing, and any subsystem that reads TOML-formatted files from the filesystem.

### Rules

- **R-TOML-001** SHOULD: Parsing errors from @iarna/toml SHOULD be caught and wrapped with context about which configuration file failed to parse.
- **R-TOML-002** MUST: All TOML parsing in configuration loading contexts MUST use @iarna/toml exclusively; no alternative TOML parsing libraries are permitted without architectural review.
- **R-TOML-003** SHOULD: Isolate TOML parsing behind clear interfaces or utility functions to enable future library substitution without widespread code changes.
- **R-TOML-004** SHOULD: Implement consistent error handling for TOML parsing failures that includes file path context and actionable error messages for configuration authors.
- **R-TOML-005** SHOULD: Follow the established pattern of parse-then-validate: use @iarna/toml for parsing, then apply schema validation separately.

### Verify

```bash
# Discover the project's dependency manifest and confirm @iarna/toml is declared as a dependency
grep -r "@iarna/toml" package.json package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null | head -5

# Search the codebase for TOML parsing import statements and verify all use @iarna/toml
grep -r "from.*toml" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" | grep -v node_modules | grep -v ".actual"

# Verify no alternative TOML parsing libraries are present
grep -E "(require|import).*\b(toml|@ltd/j-toml)\b" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" -r . 2>/dev/null | grep -v node_modules | grep -v ".actual" || echo "No alternative TOML parsers detected"

# Locate and execute the project's test suite covering configuration loading
npm test -- --testPathPattern="config|toml" 2>/dev/null || yarn test --testPathPattern="config|toml" 2>/dev/null || echo "Run configuration loading tests manually"
```

**Accept when:**
- Dependency manifest declares @iarna/toml and no alternative TOML parsing libraries are present
- All TOML parsing import statements reference @iarna/toml exclusively
- Configuration loading tests pass, demonstrating successful TOML parsing across all configuration contexts
- TOML parsing errors are caught and wrapped with file path context in all configuration loading code paths
- TOML parsing logic is isolated behind utility functions or clear interfaces

<enforcement>
Claude Code MUST NOT skip or defer verification. All TOML parsing code MUST be reviewed to confirm @iarna/toml usage and proper error wrapping with file context before accepting changes.
</enforcement>