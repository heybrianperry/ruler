# Relative Path Module Organization for First-Party Code: Runtime Built Modules Imported Bare Specifiers

These rules are ALWAYS ACTIVE for all first-party TypeScript/JavaScript modules within the project workspace, particularly imports between modules in different directories or application layers, shared utilities, type definitions, and core business logic accessed from multiple layers.

### Rules

- **R-RPMO-001** SHOULD: Runtime built-in modules SHOULD be imported using bare specifiers without path traversal.

### Verify

```bash
# Discover the project's module resolution configuration and verify no path alias mappings override relative import behavior for first-party modules
grep -r "paths\|alias" tsconfig.json jsconfig.json .eslintrc* 2>/dev/null || echo "No path alias configuration detected"

# Discover and execute the project's static analysis tooling to detect circular dependencies in the module graph
if command -v madge &> /dev/null; then madge --circular src/; fi
if command -v dpdm &> /dev/null; then dpdm --circular src/; fi

# Discover and execute the project's linting configuration to verify enforcement of relative path imports for first-party modules
if [ -f .eslintrc* ] || [ -f eslint.config.* ]; then npx eslint src/ --rule 'no-restricted-imports: [error, {patterns: ["@/*", "~/*"]}]' 2>/dev/null || true; fi
```

**Accept when:**
- All first-party module imports use relative paths with explicit directory traversal (e.g., `../`, `../../`)
- No circular dependencies exist in the module dependency graph
- Directory structure clearly separates concerns with dedicated folders for utilities (core/), types (types/), paths (paths/), and business logic (lib/)
- Runtime built-in modules are imported using bare specifiers (e.g., `import fs from 'fs'`, `import path from 'path'`)
- No path alias or absolute import mappings are configured for first-party modules

<enforcement>
Claude Code MUST NOT skip or defer verification. All first-party imports MUST be validated against relative path conventions and circular dependency detection before accepting changes.
</enforcement>