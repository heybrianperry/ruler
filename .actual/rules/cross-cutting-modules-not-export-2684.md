# Standardize Public API Contract Exports in Agent and Core Modules: Modules Not Export

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` and `src/core/` directories that expose public interfaces, as well as any module that exports types, classes, or functions consumed by other modules.

### Rules

- **R-API-001** MUST NOT: Modules MUST NOT export internal implementation details, helper functions, or private state that are not part of the intended public API.

### Verify

```bash
# Count explicit named exports (excluding default exports)
grep -r "^export " src/agents/ src/core/ | grep -v "export default" | wc -l

# Inspect exported symbols from a module
npx ts-node -e "import * as agents from './src/agents/OpenCodeAgent'; console.log(Object.keys(agents))"

# Find modules with no exports (potential internal-only modules)
find src/agents src/core -name '*.ts' -exec grep -L "^export " {} \; | grep -v ".test.ts" | grep -v ".spec.ts"

# Verify TypeScript compilation in strict mode
npx tsc --strict --noEmit
```

**Accept when:**
- All agent modules export their primary class or interface with a descriptive name
- Core utility modules explicitly list all public exports without wildcard patterns
- No internal helper functions or private implementation details appear in module export lists
- TypeScript compilation succeeds with strict mode enabled, verifying all contract types are valid

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler checks during CI build process are mandatory. Code review checklist items for API surface area review must be completed. Automated linting rules checking export patterns must pass before merge approval.
</enforcement>