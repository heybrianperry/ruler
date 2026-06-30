# Standardize Public API Contract Exports in TypeScript Modules: Class Based Contracts

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` and `src/core/` that define and export public API contracts, including classes, interfaces, and utility functions.

### Rules

- **R-EX-001** MUST: Class-based API contracts (e.g., OpenCodeAgent, ZedAgent, WarpAgent) MUST be exported as named classes with PascalCase naming.
- **R-EX-002** MUST: All public API contracts MUST use explicit named exports; default exports are prohibited except where framework compatibility requires them (documented via exception).
- **R-EX-003** MUST: Function-based utility contracts MUST be exported with camelCase naming (e.g., parseFrontmatter, loadSubagentFile).
- **R-EX-004** SHOULD: Public API contracts SHOULD be documented with JSDoc comments describing purpose, parameters, return values, and usage examples.
- **R-EX-005** SHOULD: Module files SHOULD organize public exports at the top with private implementation details below, making the public contract immediately visible.
- **R-EX-006** SHOULD: For agent implementations, the exported class name SHOULD match the file name (e.g., OpenCodeAgent in OpenCodeAgent.ts).
- **R-EX-007** MAY: Modules MAY use TypeScript's `export type` syntax for type-only exports to improve build performance and clarify intent.

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
Claude Code MUST NOT skip or defer verification of these rules. All TypeScript modules in scope MUST comply with R-EX-001 through R-EX-007 before code review approval.
</enforcement>