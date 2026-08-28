# Adopt @iarna/toml for TOML Configuration Parsing: Toml Configuration File Parsing Use Iarna

These rules are ALWAYS ACTIVE for all files that parse TOML configuration files, including agent initialization, system configuration loading, MCP server definition parsing, and any subsystem that reads TOML-formatted files from the filesystem.

### Rules

- **R-TOML-001** MUST: All TOML configuration file parsing MUST use the @iarna/toml library exclusively.
- **R-TOML-002** MUST: Isolate TOML parsing behind clear interfaces or utility functions to enable future library substitution without widespread code changes.
- **R-TOML-003** MUST: Implement consistent error handling for TOML parsing failures that includes file path context and actionable error messages for configuration authors.
- **R-TOML-004** SHOULD: Follow the established pattern of parse-then-validate: use @iarna/toml for parsing, then apply schema validation separately.
- **R-TOML-005** MUST NOT: Introduce alternative TOML parsing libraries without architectural review and engineering leadership approval.

### Verify

```bash
# Discover the project's dependency manifest and confirm @iarna/toml is declared as a dependency
grep -r "@iarna/toml" package.json package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null | head -5

# Search the codebase for TOML parsing import statements and verify all use @iarna/toml
grep -r "import.*toml" --include="*.ts" --include="*.js" --include="*.tsx" --include="*.jsx" . 2>/dev/null | grep -v node_modules | grep -v ".actual"

# Verify no alternative TOML parsing libraries are present
grep -E "(\"toml\"|'toml'|@ltd/j-toml)" package.json 2>/dev/null

# Locate and execute the project's test suite covering configuration loading
npm test -- --testPathPattern=config 2>/dev/null || yarn test --testPathPattern=config 2>/dev/null || pnpm test --testPathPattern=config 2>/dev/null
```

**Accept when:**
- Dependency manifest declares @iarna/toml and no alternative TOML parsing libraries are present
- All TOML parsing import statements reference @iarna/toml exclusively
- Configuration loading tests pass, demonstrating successful TOML parsing across all configuration contexts
- TOML parsing is wrapped in project-specific utility functions rather than called directly throughout the codebase

<enforcement>
Claude Code MUST NOT skip or defer verification. All new TOML configuration loading code MUST be reviewed to confirm @iarna/toml usage. Code review MUST reject any introduction of alternative TOML parsers without documented architectural exception.
</enforcement>