# Standardize Public API Contract Exports in Agent and Core Modules: Modules Import Dependencies

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` and `src/core/` directories that expose public interfaces, as well as any module that exports types, classes, or functions consumed by other modules.

### Rules

- **R-API-001** MUST: Modules MUST import dependencies using explicit relative or package paths (e.g., `./IAgent`, `../core/FileSystemUtils`, `fs/promises`) to maintain clear dependency graphs.

### Verify

```bash
# Count explicit exports across agent and core modules
grep -r "^export " src/agents/ src/core/ | grep -v "export default" | wc -l

# Verify named exports from a sample agent module
npx ts-node -e "import * as agents from './src/agents/OpenCodeAgent'; console.log(Object.keys(agents))"

# Find modules without explicit exports (excluding test files)
find src/agents src/core -name '*.ts' -exec grep -L "^export " {} \; | grep -v ".test.ts" | grep -v ".spec.ts"
```

**Accept when:**
- All agent modules export their primary class or interface with a descriptive name
- Core utility modules explicitly list all public exports without wildcard patterns
- No internal helper functions or private implementation details appear in module export lists
- TypeScript compilation succeeds with strict mode enabled, verifying all contract types are valid
- All imports use explicit relative or package paths (no implicit or ambiguous import resolution)

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler checks during CI build process are mandatory. Code review must verify API surface area. Automated linting rules checking export patterns must pass before merge approval.
</enforcement>