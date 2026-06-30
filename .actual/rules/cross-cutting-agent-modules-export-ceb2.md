# Standardize Public API Contract Exports in Agent and Core Modules: Agent Modules Export

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` and `src/core/` directories that expose public interfaces, including interface definitions and type declarations intended for cross-module usage.

### Rules

- **R-API-001** MUST: All agent modules MUST export a named class or interface representing their primary public contract (e.g., OpenCodeAgent, ZedAgent).
- **R-API-002** MUST: Core utility modules MUST explicitly list all public exports without wildcard patterns.
- **R-API-003** MUST: No internal helper functions or private implementation details MUST appear in module export lists.
- **R-API-004** SHOULD: Use TypeScript's `export` keyword explicitly for each public contract rather than collecting exports at the end of files.
- **R-API-005** SHOULD: Consider creating `index.ts` files in directories to aggregate and re-export public contracts from multiple modules.
- **R-API-006** SHOULD: Document public contracts with JSDoc comments explaining their purpose, usage, and any stability guarantees.
- **R-API-007** SHOULD: Use TypeScript's `export type` for type-only exports to enable better tree-shaking and clarify intent.
- **R-API-008** SHOULD: Establish naming conventions for exported contracts (e.g., Agent suffix for agent classes, Mapping suffix for configuration types).

### Verify

```bash
# Count explicit exports in agent and core modules
grep -r "^export " src/agents/ src/core/ | grep -v "export default" | wc -l

# Verify exports from a specific agent module
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
- No wildcard exports (`export *`) are present in agent or core modules

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler checks during CI build process are mandatory. Code review checklist items for API surface area review are required. Automated linting rules checking export patterns must pass before merge approval.
</enforcement>