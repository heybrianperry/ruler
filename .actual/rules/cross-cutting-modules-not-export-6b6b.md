# Standardize Public API Contract Exports in TypeScript Modules: Modules Not Export

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` and `src/core/` that define public API contracts including classes, interfaces, and utility functions.

### Rules

- **R-EXPORT-001** MUST NOT: Modules MUST NOT export internal implementation details, private utilities, or temporary data structures as part of the public contract.
- **R-EXPORT-002** MUST: Use explicit named exports for all public API contracts rather than default exports, except where framework compatibility requires otherwise (documented via exception).
- **R-EXPORT-003** MUST: Class exports MUST follow PascalCase naming convention (e.g., `OpenCodeAgent`, `AbstractAgent`).
- **R-EXPORT-004** MUST: Function exports MUST follow camelCase naming convention (e.g., `parseFrontmatter`, `loadSubagentFile`).
- **R-EXPORT-005** SHOULD: Document public API contracts with JSDoc comments describing purpose, parameters, return values, and usage examples.
- **R-EXPORT-006** SHOULD: Organize module files with public exports at the top and private implementation details below for immediate visibility of the public contract.
- **R-EXPORT-007** MAY: Use TypeScript's `export type` syntax for type-only exports to improve build performance and clarify intent.

### Verify

```bash
# Verify no default exports in agent and core modules (except framework-mandated exceptions)
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
- Verification commands execute without detecting default exports or naming convention violations

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All modules in scope MUST comply with R-EXPORT-001 through R-EXPORT-007 before code review approval.
</enforcement>