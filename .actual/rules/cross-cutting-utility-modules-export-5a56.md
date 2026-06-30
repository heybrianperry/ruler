# Standardize Public API Contract Exports in Agent and Core Modules: Utility Modules Export

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` and `src/core/` directories that expose public interfaces for cross-module consumption.

### Rules

- **R-API-001** MAY: Utility modules MAY export multiple related contracts as a cohesive API surface when they represent a logical grouping of functionality.

### Verify

```bash
# Count explicit named exports across agent and core modules
grep -r "^export " src/agents/ src/core/ | grep -v "export default" | wc -l

# Verify primary exports from OpenCodeAgent module
npx ts-node -e "import * as agents from './src/agents/OpenCodeAgent'; console.log(Object.keys(agents))"

# Find modules without any exports (excluding test files)
find src/agents src/core -name '*.ts' -exec grep -L "^export " {} \; | grep -v ".test.ts" | grep -v ".spec.ts"
```

**Accept when:**
- All agent modules export their primary class or interface with a descriptive name
- Core utility modules explicitly list all public exports without wildcard patterns
- No internal helper functions or private implementation details appear in module export lists
- TypeScript compilation succeeds with strict mode enabled, verifying all contract types are valid

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler checks during CI build process are mandatory. Code review checklist items for API surface area review must be completed. Automated linting rules checking export patterns must pass before merge approval.
</enforcement>