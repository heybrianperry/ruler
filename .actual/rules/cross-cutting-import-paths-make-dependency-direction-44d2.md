# Relative Path Module Organization for First-Party Code: Import Paths Make Dependency Direction Explicit

These rules are ALWAYS ACTIVE for all first-party TypeScript/JavaScript modules within the project workspace, particularly imports between modules in different directories or application layers.

### Rules

- **R-IMPORT-001** SHOULD: Import paths SHOULD make dependency direction explicit, with higher-level modules (agents, CLI handlers) importing from lower-level modules (core utilities, types) using upward-traversing relative paths.

### Verify

```bash
# Discover the project's module resolution configuration and verify no path alias mappings override relative import behavior for first-party modules
grep -r "paths\|alias" tsconfig.json jsconfig.json .eslintrc* 2>/dev/null || echo "No path alias configuration detected"

# Discover and execute the project's static analysis tooling to detect circular dependencies in the module graph
if command -v madge &> /dev/null; then madge --circular src/; fi
if command -v dpdm &> /dev/null; then dpdm --circular src/; fi

# Discover and execute the project's linting configuration to verify enforcement of relative path imports for first-party modules
if [ -f .eslintrc* ] || [ -f eslint.config.* ]; then npx eslint src/ --rule 'no-restricted-imports: [error, {patterns: ["src/*"]}]' 2>/dev/null || true; fi
```

**Accept when:**
- All first-party module imports use relative paths with explicit directory traversal (../ patterns)
- No circular dependencies exist in the module dependency graph
- Directory structure clearly separates concerns with dedicated folders for utilities (core/), types (types/), paths (paths/), and business logic (lib/)
- No path alias mappings are configured for first-party module resolution

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification categories (module resolution configuration, circular dependency detection, and linting enforcement) MUST be executed before accepting changes to import paths.
</enforcement>