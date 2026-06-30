# Standardize Public API Contract Exports in TypeScript Modules: Interface Type Definitions

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` and `src/core/` that define public API contracts, including classes, interfaces, and utility functions.

### Rules

- **R-API-001** MUST: Interface and type definitions that form part of the public contract (e.g., ParsedFrontmatter, CopilotToolMapping) MUST be exported alongside their implementing classes or consuming functions.
- **R-API-002** MUST: Use explicit named exports for all public contracts rather than default exports, except where framework compatibility requires otherwise (documented via exception).
- **R-API-003** MUST: Export class names using PascalCase and function names using camelCase consistently across all modules.
- **R-API-004** SHOULD: Document public API contracts with JSDoc comments describing purpose, parameters, return values, and usage examples.
- **R-API-005** SHOULD: Organize module files with public exports at the top and private implementation details below, making the public contract immediately visible.
- **R-API-006** MAY: Use TypeScript's `export type` syntax for type-only exports to improve build performance and clarify intent.

### Verify

```bash
# Verify no default exports in agent and core modules (except framework-mandated exceptions)
grep -r "export default" src/agents/ src/core/ | grep -v ".test.ts" | grep -v ".spec.ts"

# Verify PascalCase class exports
grep -r "export class [A-Z]" src/agents/ src/core/

# Verify camelCase function exports
grep -r "export function [a-z]" src/agents/ src/core/

# Verify imports resolve successfully
npx ts-node -e "import * as agents from './src/agents'; import * as core from './src/core'; console.log('Imports successful')"
```

**Accept when:**
- All TypeScript modules in `src/agents/` and `src/core/` use explicit named exports with no default exports (except framework-mandated exceptions documented in code comments)
- Class exports follow PascalCase naming and function exports follow camelCase naming consistently across all modules
- Public API contracts are documented with JSDoc comments and can be imported successfully by dependent modules
- Automated linting passes with no violations of export naming conventions or unintended public contract exports
- Verification commands execute without detecting default exports or naming convention violations

<enforcement>
Claude Code MUST NOT skip or defer verification. All verification commands MUST execute successfully before accepting changes to TypeScript modules in scope. ESLint rules (no-default-export, naming-convention) and TypeScript strict mode MUST pass. Code review MUST verify public contract documentation and naming compliance. CI pipeline MUST run verification commands and block merges on violations.
</enforcement>