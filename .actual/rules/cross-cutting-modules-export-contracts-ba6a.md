# Standardize Public API Contract Exports in TypeScript Modules: Modules Export Contracts

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` and `src/core/` that define and export public API contracts including classes, interfaces, and utility functions.

### Rules

- **R-EXP-001** MUST: Use explicit named exports for all public API contracts; default exports are prohibited except where framework requirements mandate them (EXC-001).
- **R-EXP-002** MUST: Name exported classes using PascalCase (e.g., `OpenCodeAgent`, `AbstractAgent`) and ensure the exported class name matches the file name.
- **R-EXP-003** MUST: Name exported functions using camelCase (e.g., `parseFrontmatter`, `loadSubagentFile`, `resolveSelectedAgents`).
- **R-EXP-004** MUST: Document all public API contracts with JSDoc comments describing purpose, parameters, return values, and usage examples.
- **R-EXP-005** SHOULD: Use TypeScript's `export type` syntax for type-only exports to improve build performance and clarify intent.
- **R-EXP-006** SHOULD: Organize module files with public exports at the top and private implementation details below, making the public contract immediately visible.
- **R-EXP-007** MAY: Modules MAY re-export contracts from dependencies to create facade patterns or simplified public APIs.

### Verify

```bash
# Verify no default exports exist (except framework-mandated exceptions)
grep -r "export default" src/agents/ src/core/ | grep -v ".test.ts" | grep -v ".spec.ts"

# Verify PascalCase class exports
grep -r "export class [A-Z]" src/agents/ src/core/

# Verify camelCase function exports
grep -r "export function [a-z]" src/agents/ src/core/

# Verify imports resolve successfully
npx ts-node -e "import * as agents from './src/agents'; import * as core from './src/core'; console.log('Imports successful')"
```

**Accept when:**
- All TypeScript modules in `src/agents/` and `src/core/` use explicit named exports with no default exports (except framework-mandated exceptions documented with EXC-001).
- Class exports follow PascalCase naming and function exports follow camelCase naming consistently across all modules.
- Public API contracts are documented with JSDoc comments and can be imported successfully by dependent modules.
- Automated linting passes with no violations of export naming conventions or unintended public contract exports.
- Verification commands return no unexpected default exports and confirm successful module imports.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for TypeScript modules in the specified scope. Violations must be corrected before code review approval.
</enforcement>