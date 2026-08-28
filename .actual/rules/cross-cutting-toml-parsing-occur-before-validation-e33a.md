# Adopt @iarna/toml for TOML Configuration Parsing: Toml Parsing Occur Before Validation Logic

These rules are ALWAYS ACTIVE for all configuration file loading code where the source format is TOML, including agent configuration initialization, system configuration loading, MCP server definition parsing, and any subsystem that reads TOML-formatted files from the filesystem.

### Rules

- **R-TOML-001** MUST: TOML parsing MUST occur before validation logic to separate parsing concerns from schema validation.
- **R-TOML-002** MUST: All TOML parsing in configuration loading contexts MUST use @iarna/toml exclusively; no alternative TOML parsing libraries are permitted without architectural review and exception approval.
- **R-TOML-003** MUST: TOML parsing logic MUST be isolated behind clear interfaces or utility functions to enable future library substitution without widespread code changes.
- **R-TOML-004** MUST: TOML parsing failures MUST include file path context and actionable error messages for configuration authors.
- **R-TOML-005** SHOULD: When adding new configuration files, follow the established pattern of parse-then-validate: use @iarna/toml for parsing, then apply schema validation separately.

### Verify

```bash
# Discover the project's dependency manifest and confirm @iarna/toml is declared as a dependency
grep -r "@iarna/toml" package.json package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null || echo "Dependency manifest check required"

# Search the codebase for TOML parsing import statements and verify all use @iarna/toml with no alternative TOML parsers present
grep -r "from.*toml" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" . | grep -v node_modules | grep -v ".actual"

# Verify no alternative TOML parsing libraries are imported
grep -r "require.*toml\|import.*toml" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" . | grep -v node_modules | grep -v ".actual" | grep -v "@iarna/toml"

# Locate and execute the project's test suite covering configuration loading
npm test -- --testPathPattern=config 2>/dev/null || yarn test --testPathPattern=config 2>/dev/null || echo "Test execution required"
```

**Accept when:**
- Dependency manifest declares @iarna/toml and no alternative TOML parsing libraries are present
- All TOML parsing import statements reference @iarna/toml exclusively
- Configuration loading tests pass, demonstrating successful TOML parsing across all configuration contexts
- TOML parsing is wrapped in utility functions or clear interfaces rather than called directly throughout the codebase
- TOML parsing errors include file path context and actionable messages

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review rejection is required if alternative TOML parsers are introduced without architectural review. CI pipeline failure occurs if dependency audit detects multiple TOML parsing libraries. Refactoring is required for code that bypasses the standard TOML parsing approach.
</enforcement>