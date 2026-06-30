# Standardize Public API Contract Exports in Agent and Core Modules: Public Contract Exports

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` and `src/core/` directories that expose public interfaces for cross-module consumption.

### Rules

- **R-PUB-001** SHOULD: Public contract exports SHOULD be documented with TypeScript types or interfaces to provide compile-time contract verification.
- **R-PUB-002** MUST: All agent modules MUST export their primary class or interface with a descriptive name (e.g., `OpenCodeAgent`, `ZedAgent`).
- **R-PUB-003** MUST: Core utility modules MUST explicitly list all public exports without wildcard patterns (`export *`).
- **R-PUB-004** MUST: No internal helper functions or private implementation details MUST appear in module export lists.
- **R-PUB-005** SHOULD: Use TypeScript's `export` keyword explicitly for each public contract rather than collecting exports at the end of files.
- **R-PUB-006** SHOULD: Consider creating `index.ts` files in directories to aggregate and re-export public contracts from multiple modules.
- **R-PUB-007** SHOULD: Document public contracts with JSDoc comments explaining their purpose, usage, and any stability guarantees.
- **R-PUB-008** SHOULD: Use TypeScript's `export type` for type-only exports to enable better tree-shaking and clarify intent.
- **R-PUB-009** MAY: Establish naming conventions for exported contracts (e.g., Agent suffix for agent classes, Mapping suffix for configuration types).

### Verify

```bash
# Count explicit named exports (excluding default exports)
grep -r "^export " src/agents/ src/core/ | grep -v "export default" | wc -l

# Inspect exported symbols from a module
npx ts-node -e "import * as agents from './src/agents/OpenCodeAgent'; console.log(Object.keys(agents))"

# Find modules without any exports (excluding test files)
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
Claude Code MUST NOT skip or defer verification. TypeScript compiler checks during CI build process are mandatory. Code review checklist items for API surface area review are required. Automated linting rules checking export patterns must pass. Violations result in CI build failure and require explicit justification in code review.
</enforcement>