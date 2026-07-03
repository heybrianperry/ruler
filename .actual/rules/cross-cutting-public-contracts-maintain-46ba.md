# Standardize Public API Contract Exports in TypeScript Modules: Public Contracts Maintain

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` and `src/core/` that define and export public API contracts, including classes, interfaces, and utility functions.

### Rules

- **R-PUB-001** MUST: Public API contracts MUST maintain stable signatures across versions, with breaking changes documented and versioned appropriately.
- **R-PUB-002** MUST: All TypeScript modules in `src/agents/` and `src/core/` MUST use explicit named exports with no default exports, except where framework compatibility requires it (documented via exception).
- **R-PUB-003** MUST: Class exports MUST follow PascalCase naming convention (e.g., `OpenCodeAgent`, `AbstractAgent`).
- **R-PUB-004** MUST: Function exports MUST follow camelCase naming convention (e.g., `parseFrontmatter`, `resolveSelectedAgents`).
- **R-PUB-005** SHOULD: Public API contracts SHOULD be documented with JSDoc comments describing purpose, parameters, return values, and usage examples.
- **R-PUB-006** SHOULD: Module files SHOULD organize public exports at the top with private implementation details below, making the public contract immediately visible.
- **R-PUB-007** SHOULD: For agent implementations, the exported class name SHOULD match the file name (e.g., `OpenCodeAgent` in `OpenCodeAgent.ts`).
- **R-PUB-008** MAY: Developers MAY use TypeScript's `export type` syntax for type-only exports to improve build performance and clarify intent.

### Verify

```bash
# Verify no default exports in agent and core modules
grep -r "export default" src/agents/ src/core/ | grep -v ".test.ts" | grep -v ".spec.ts"

# Verify PascalCase class exports
grep -r "export class [A-Z]" src/agents/ src/core/

# Verify camelCase function exports
grep -r "export function [a-z]" src/agents/ src/core/

# Verify imports are successful
npx ts-node -e "import * as agents from './src/agents'; import * as core from './src/core'; console.log('Imports successful')"
```

**Accept when:**
- All TypeScript modules in `src/agents/` and `src/core/` use explicit named exports with no default exports (except framework-mandated exceptions documented in code comments)
- Class exports follow PascalCase naming and function exports follow camelCase naming consistently across all modules
- Public API contracts are documented with JSDoc comments and can be imported successfully by dependent modules
- Automated linting passes with no violations of export naming conventions or unintended public contract exports
- ESLint rules enforcing named exports and naming conventions report no violations
- TypeScript compiler in strict mode catches no type mismatches across module boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. All verify commands MUST execute successfully before accepting changes to public API contracts. ESLint and TypeScript compiler errors MUST be resolved. Code review MUST verify public contract documentation and naming conventions.
</enforcement>