# Standardize Public API Contract Exports in TypeScript Modules: Function Based Contracts

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` and `src/core/` that define and export public API contracts, including function-based utilities, class-based implementations, and interface definitions.

### Rules

- **R-EXPORT-001** MUST: Function-based API contracts (e.g., parseFrontmatter, loadSubagentFile, mapRawAgentConfigs) MUST be exported as named functions with camelCase naming.
- **R-EXPORT-002** MUST: Class-based API contracts MUST be exported as named classes with PascalCase naming.
- **R-EXPORT-003** MUST: All public API contracts MUST use explicit named exports (not default exports), except where framework compatibility requires otherwise (see exceptions).
- **R-EXPORT-004** MUST: Public API contracts MUST be documented with JSDoc comments describing purpose, parameters, return values, and usage examples.
- **R-EXPORT-005** SHOULD: Organize module files with public exports at the top and private implementation details below, making the public contract immediately visible.
- **R-EXPORT-006** SHOULD: Use TypeScript's `export type` syntax for type-only exports to improve build performance and clarify intent.
- **R-EXPORT-007** MAY: A module may export a default export for framework compatibility (e.g., Next.js page components, React components), provided a named export is maintained alongside it.

### Verify

```bash
# Verify no default exports in agent and core modules (except framework-mandated exceptions)
grep -r "export default" src/agents/ src/core/ | grep -v ".test.ts" | grep -v ".spec.ts"

# Verify class exports use PascalCase
grep -r "export class [A-Z]" src/agents/ src/core/

# Verify function exports use camelCase
grep -r "export function [a-z]" src/agents/ src/core/

# Verify imports work successfully across modules
npx ts-node -e "import * as agents from './src/agents'; import * as core from './src/core'; console.log('Imports successful')"
```

**Accept when:**
- All TypeScript modules in `src/agents/` and `src/core/` use explicit named exports with no default exports (except framework-mandated exceptions documented in code comments)
- Class exports follow PascalCase naming and function exports follow camelCase naming consistently across all modules
- Public API contracts are documented with JSDoc comments and can be imported successfully by dependent modules
- Automated linting passes with no violations of export naming conventions or unintended public contract exports
- Verification commands execute without detecting default exports or naming convention violations

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All public API contracts in scope MUST comply with R-EXPORT-001 through R-EXPORT-007 before code is considered complete.
</enforcement>