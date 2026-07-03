# Standardize Public API Contract Exports in TypeScript Modules: Typescript Modules That

These rules are ALWAYS ACTIVE for all TypeScript modules that define public API contracts within the codebase, including agent implementations in `src/agents/` and shared utilities in `src/core/`.

### Rules

- **R-EXP-001** MUST: All TypeScript modules that define public API contracts MUST use explicit named exports rather than default exports.
- **R-EXP-002** MUST: Class exports MUST follow PascalCase naming conventions (e.g., `OpenCodeAgent`, `AbstractAgent`).
- **R-EXP-003** MUST: Function exports MUST follow camelCase naming conventions (e.g., `parseFrontmatter`, `loadSubagentFile`).
- **R-EXP-004** MUST: Public API contracts MUST be documented with JSDoc comments describing purpose, parameters, return values, and usage examples.
- **R-EXP-005** SHOULD: Organize module files with public exports at the top and private implementation details below for immediate visibility of the public contract.
- **R-EXP-006** SHOULD: Use TypeScript's `export type` syntax for type-only exports to improve build performance and clarify intent.
- **R-EXP-007** MAY: A module may export a default export only when required for framework compatibility (e.g., Next.js page components, React components), provided a named export is maintained alongside it and documented with a reference to this rule.

### Verify

```bash
# Verify no default exports in agent and core modules (excluding tests)
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
- Verification commands return no results for default exports (excluding documented exceptions) and successfully resolve all imports

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All TypeScript modules in scope MUST be checked for compliance with R-EXP-001 through R-EXP-007 before code review approval.
</enforcement>