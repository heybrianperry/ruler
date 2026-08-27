# Relative Path Module Organization for First-Party Code: Shared Utilities Type Definitions Core Business

These rules are ALWAYS ACTIVE for all first-party TypeScript/JavaScript module imports within the project workspace, particularly for imports between modules in different directories or application layers accessing shared utilities, type definitions, and core business logic.

### Rules

- **R-RPMO-001** MUST: Organize shared utilities, type definitions, and core business logic in dedicated directories that establish clear module ownership boundaries (core/, types/, paths/, lib/).
- **R-RPMO-002** MUST: Use relative path imports with explicit directory traversal for all first-party module imports between different directories or application layers.
- **R-RPMO-003** MUST: Construct relative import paths by counting directory levels upward to the common ancestor, then downward to the target module.
- **R-RPMO-004** MUST: Never use path alias configuration or absolute-style imports for first-party modules; maintain explicit relative path visibility.
- **R-RPMO-005** SHOULD: Use IDE features for automatic import path generation to reduce manual path construction errors and verify generated paths follow the relative convention.
- **R-RPMO-006** SHOULD: Establish directory structure early and document the convention explicitly to ensure consistent application across the codebase.
- **R-RPMO-007** MAY: Defer barrel exports (index files) until public API surface needs to be distinguished from internal implementation details within a module directory.

### Verify

```bash
# Discover the project's module resolution configuration and verify no path alias mappings override relative import behavior for first-party modules
grep -r "paths\|alias" tsconfig.json jsconfig.json 2>/dev/null || echo "No path alias configuration detected"

# Discover and execute the project's static analysis tooling to detect circular dependencies in the module graph
if [ -f "package.json" ]; then
  grep -E "madge|depcheck|circular" package.json || echo "No circular dependency detection tool configured"
fi

# Discover and execute the project's linting configuration to verify enforcement of relative path imports for first-party modules
if [ -f ".eslintrc" ] || [ -f ".eslintrc.json" ] || [ -f ".eslintrc.js" ]; then
  echo "ESLint configuration found"
  grep -r "import" .eslintrc* 2>/dev/null || echo "No import-related rules configured"
fi

# Verify all first-party imports use relative paths
find src -name "*.ts" -o -name "*.js" | xargs grep -E "^import.*from ['\"](?!\\.|/|@)" 2>/dev/null | grep -v node_modules || echo "No non-relative first-party imports detected"
```

**Accept when:**
- All first-party module imports use relative paths with explicit directory traversal (../ or ./)
- No circular dependencies exist in the module dependency graph
- Directory structure clearly separates concerns with dedicated folders for utilities (core/), types (types/), paths (paths/), and business logic (lib/)
- No path alias configuration overrides relative import behavior for first-party modules
- Linting rules enforce relative path usage for first-party imports

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and pull request acceptance. Circular dependency detection and relative path enforcement MUST be verified before merge.
</enforcement>