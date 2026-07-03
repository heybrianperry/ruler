# Standardize Public API Contract Exports in Agent and Core Modules: Agent Implementations Implement

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` and `src/core/` directories that expose public interfaces, including agent implementations, core utility modules, and any module that exports types, classes, or functions consumed by other modules.

### Rules

- **R-API-001** SHOULD: Agent implementations SHOULD implement or extend common base interfaces (e.g., IAgent) to ensure consistent contract shapes across agent types.
- **R-API-002** SHOULD: Use TypeScript's `export` keyword explicitly for each public contract rather than collecting exports at the end of files.
- **R-API-003** SHOULD: Create `index.ts` files in directories to aggregate and re-export public contracts from multiple modules.
- **R-API-004** SHOULD: Document public contracts with JSDoc comments explaining their purpose, usage, and any stability guarantees.
- **R-API-005** SHOULD: Use TypeScript's `export type` for type-only exports to enable better tree-shaking and clarify intent.
- **R-API-006** SHOULD: Establish naming conventions for exported contracts (e.g., Agent suffix for agent classes, Mapping suffix for configuration types).
- **R-API-007** MUST: Not expose internal helper functions or private implementation details in module export lists.
- **R-API-008** MUST: Not use wildcard exports (`export *`) for public API contracts; use explicit named exports instead.

### Verify

```bash
# Count explicit exports in agent and core modules
grep -r "^export " src/agents/ src/core/ | grep -v "export default" | wc -l

# Verify agent module exports are discoverable
npx ts-node -e "import * as agents from './src/agents/OpenCodeAgent'; console.log(Object.keys(agents))"

# Find modules without any exports (potential issues)
find src/agents src/core -name '*.ts' -exec grep -L "^export " {} \; | grep -v ".test.ts" | grep -v ".spec.ts"

# Verify TypeScript compilation in strict mode
npx tsc --strict --noEmit
```

**Accept when:**
- All agent modules export their primary class or interface with a descriptive name
- Core utility modules explicitly list all public exports without wildcard patterns
- No internal helper functions or private implementation details appear in module export lists
- TypeScript compilation succeeds with strict mode enabled, verifying all contract types are valid
- All public exports are documented with JSDoc comments
- Named exports follow established naming conventions (Agent suffix, Mapping suffix, etc.)

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler checks during CI build process are mandatory. Code review must include explicit API surface area review. Linting rules checking export patterns must pass before merge approval.
</enforcement>