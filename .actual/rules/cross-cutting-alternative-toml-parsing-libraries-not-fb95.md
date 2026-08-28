# Adopt @iarna/toml for TOML Configuration Parsing: Alternative Toml Parsing Libraries Not Introduced

These rules are ALWAYS ACTIVE for all files that parse TOML configuration files, including agent initialization, system configuration loading, MCP server definition parsing, and any subsystem that reads TOML-formatted files from the filesystem.

### Rules

- **R-TOML-001** MUST NOT: Alternative TOML parsing libraries MUST NOT be introduced without explicit architectural review and migration plan.
- **R-TOML-002** MUST: All TOML configuration file parsing MUST use @iarna/toml exclusively across all configuration subsystems.
- **R-TOML-003** MUST: TOML parsing MUST be isolated behind clear interfaces or utility functions to enable future library substitution without widespread code changes.
- **R-TOML-004** MUST: Consistent error handling for TOML parsing failures MUST include file path context and actionable error messages for configuration authors.
- **R-TOML-005** SHOULD: Follow the established pattern of parse-then-validate: use @iarna/toml for parsing, then apply schema validation separately.

### Verify

```bash
# Discover the project's dependency manifest and confirm @iarna/toml is declared as a dependency
grep -r "@iarna/toml" package.json package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null || echo "Dependency manifest check required"

# Search the codebase for TOML parsing import statements and verify all use @iarna/toml
grep -r "from.*toml" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" . | grep -v node_modules | grep -v ".actual"

# Verify no alternative TOML parsing libraries are imported
grep -r "require.*toml\|import.*toml" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" . | grep -v node_modules | grep -v ".actual" | grep -v "@iarna/toml"

# Locate and execute the project's test suite covering configuration loading
npm test -- --testPathPattern="config|toml" 2>/dev/null || echo "Test suite execution required"
```

**Accept when:**
- Dependency manifest declares @iarna/toml and no alternative TOML parsing libraries are present
- All TOML parsing import statements reference @iarna/toml exclusively
- Configuration loading tests pass, demonstrating successful TOML parsing across all configuration contexts
- No grep results show alternative TOML parsing libraries in use

<enforcement>
Code review MUST reject any pull request introducing alternative TOML parsing libraries without documented architectural review and migration plan. CI pipeline MUST fail if dependency audit detects multiple TOML parsing libraries. Static analysis or linting rules MUST enforce import patterns for TOML parsing. Claude Code MUST NOT skip or defer verification of these rules.
</enforcement>