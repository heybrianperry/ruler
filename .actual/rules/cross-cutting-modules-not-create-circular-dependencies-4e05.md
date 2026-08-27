# Relative Path Module Organization for First-Party Code: Modules Not Create Circular Dependencies Relative

These rules are ALWAYS ACTIVE for all first-party TypeScript/JavaScript modules within the project workspace, particularly imports between modules in different directories or application layers, shared utilities, type definitions, and core business logic accessed from multiple layers.

### Rules

- **R-CIRC-001** MUST NOT: Modules MUST NOT create circular dependencies; the relative path structure MUST enforce a directed acyclic graph of module dependencies.
- **R-CIRC-002** MUST: All first-party module imports MUST use relative paths with explicit directory traversal (e.g., `../`, `../../`).
- **R-CIRC-003** MUST: When creating new modules, place them in the appropriate directory based on their architectural role: shared utilities in `core/`, type definitions in `types/`, domain-specific paths in `paths/`, and reusable business logic in `lib/`.
- **R-CIRC-004** SHOULD: Use IDE features for automatic import path generation to reduce manual path construction errors; verify generated paths follow the relative convention rather than introducing aliases.
- **R-CIRC-005** MUST NOT: First-party imports MUST NOT use path alias mappings or absolute-style imports that override relative import behavior.

### Verify

```bash
# Discover the project's module resolution configuration and verify no path alias mappings override relative import behavior for first-party modules
grep -r "paths\|alias" tsconfig.json jsconfig.json .eslintrc* 2>/dev/null || echo "No path alias configuration detected"

# Discover and execute the project's static analysis tooling to detect circular dependencies in the module graph
if command -v madge &> /dev/null; then
  madge --circular src/
fi

# Discover and execute the project's linting configuration to verify enforcement of relative path imports for first-party modules
if [ -f .eslintrc.json ] || [ -f .eslintrc.js ] || [ -f .eslintrc.yml ]; then
  npm run lint -- --format json 2>/dev/null | grep -i "import\|relative" || echo "Linting complete"
fi

# Verify all first-party imports use relative paths
grep -r "from ['\"]\([^./]\|@\)" src/ --include="*.ts" --include="*.js" | grep -v node_modules | grep -v "from ['\"]@types" || echo "No absolute first-party imports detected"
```

**Accept when:**
- All first-party module imports use relative paths with explicit directory traversal
- No circular dependencies exist in the module dependency graph
- Directory structure clearly separates concerns with dedicated folders for utilities, types, and business logic
- Static analysis tools confirm no path alias mappings override relative import behavior for first-party modules
- Linting configuration enforces relative path imports for first-party modules

<enforcement>
Claude Code MUST NOT skip or defer verification. Circular dependency detection failures halt the build and require immediate resolution. Automated linting failures block pull request merges when non-relative imports are detected for first-party modules.
</enforcement>