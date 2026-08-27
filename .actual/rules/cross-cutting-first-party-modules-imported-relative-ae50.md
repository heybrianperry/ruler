# Relative Path Module Organization for First-Party Code: First Party Modules Imported Relative Paths

These rules are ALWAYS ACTIVE for all first-party TypeScript/JavaScript modules within the project workspace, particularly imports between modules in different directories or application layers, shared utilities, type definitions, and core business logic accessed from multiple layers.

### Rules

- **R-RPMO-001** MUST: First-party modules MUST be imported using relative paths that explicitly traverse the directory structure from the importing module to the target module.

### Verify

```bash
# Discover the project's module resolution configuration and verify no path alias mappings override relative import behavior for first-party modules
find . -name "tsconfig.json" -o -name "jsconfig.json" -o -name ".eslintrc*" | xargs grep -l "paths\|alias" 2>/dev/null || echo "No path alias configuration detected"

# Discover and execute the project's static analysis tooling to detect circular dependencies in the module graph
if [ -f "package.json" ]; then
  grep -E '"(madge|depcheck|circular-dependency-plugin)"' package.json && madge --circular src/ || echo "Circular dependency detection tool not configured"
fi

# Discover and execute the project's linting configuration to verify enforcement of relative path imports for first-party modules
if [ -f ".eslintrc.json" ] || [ -f ".eslintrc.js" ] || [ -f "eslint.config.js" ]; then
  grep -r "import.*from ['\"]\./" src/ | head -20 || echo "No relative imports found"
fi
```

**Accept when:**
- All first-party module imports use relative paths with explicit directory traversal (e.g., `../`, `../../`)
- No circular dependencies exist in the module dependency graph
- Directory structure clearly separates concerns with dedicated folders for utilities, types, and business logic (core/, types/, paths/, lib/)
- Static analysis confirms no path alias mappings override relative import behavior for first-party modules
- Code review verification confirms import path consistency across the codebase

<enforcement>
Claude Code MUST NOT skip or defer verification. All first-party imports must be validated against R-RPMO-001 before accepting changes. Circular dependency detection MUST be run in the verification phase. Linting rules enforcing relative path usage MUST be confirmed as active in the project configuration.
</enforcement>