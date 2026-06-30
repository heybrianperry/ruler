# Standardize Public API Contract Exports in TypeScript Modules: Modules Group Related

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` and `src/core/` that define and export public API contracts.

### Rules

- **R-MOD-001** SHOULD: Modules SHOULD group related exports by functional domain (e.g., agent implementations, configuration utilities, file system operations).

### Verify

```bash
# Verify no default exports in agent and core modules (except framework-mandated exceptions)
grep -r "export default" src/agents/ src/core/ | grep -v ".test.ts" | grep -v ".spec.ts"

# Verify class exports follow PascalCase naming
grep -r "export class [A-Z]" src/agents/ src/core/

# Verify function exports follow camelCase naming
grep -r "export function [a-z]" src/agents/ src/core/

# Verify imports resolve successfully
npx ts-node -e "import * as agents from './src/agents'; import * as core from './src/core'; console.log('Imports successful')"
```

**Accept when:**
- All TypeScript modules in `src/agents/` and `src/core/` use explicit named exports with no default exports (except framework-mandated exceptions documented with JSDoc)
- Class exports follow PascalCase naming and function exports follow camelCase naming consistently across all modules
- Public API contracts are documented with JSDoc comments describing purpose, parameters, return values, and usage examples
- Automated linting passes with no violations of export naming conventions or unintended public contract exports
- All cross-module imports resolve successfully without errors

<enforcement>
Claude Code MUST NOT skip or defer verification. ESLint rules (no-default-export, naming-convention) and TypeScript strict mode MUST pass before accepting changes. CI pipeline MUST run verification commands and block pull requests on violations.
</enforcement>