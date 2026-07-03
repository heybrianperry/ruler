# Standardize Public API Contract Exports via Named Class/Interface Patterns: Modules Not Expose

These rules are ALWAYS ACTIVE for all TypeScript modules in src/agents/, src/core/, src/paths/, and src/vscode/ that export public API contracts, as well as any new modules that provide reusable functionality to other parts of the codebase.

### Rules

- **R-API-001** MUST_NOT: Modules MUST NOT expose internal implementation details (private helper functions, internal state management) through public exports.
- **R-API-002** MUST: Agent modules MUST export named classes implementing the IAgent interface.
- **R-API-003** MUST: Configuration and utility modules MUST export named interfaces/types alongside implementation functions.
- **R-API-004** SHOULD: Use TypeScript's 'export { ... }' syntax at the end of files to create a clear manifest of public exports.
- **R-API-005** SHOULD: Document public interfaces with JSDoc comments explaining their purpose, usage constraints, and behavioral guarantees.
- **R-API-006** MAY: Use default exports only in documented exception cases (single utility functions with no associated types or configuration).

### Verify

```bash
# Count named exports across public API modules
grep -r 'export class\|export interface\|export type' src/agents/ src/core/ src/paths/ src/vscode/ | wc -l

# Verify minimal default exports in public modules
grep -r 'export default' src/agents/ src/core/ | grep -v '.test.ts' | wc -l

# Count agent classes exported
npx ts-node -e "import * as agents from './src/agents'; console.log(Object.keys(agents).filter(k => k.includes('Agent')).length)"
```

**Accept when:**
- All agent modules export named classes implementing IAgent interface
- Configuration and utility modules export named interfaces/types alongside implementation functions
- Default exports are used only in documented exception cases (single utility functions)
- Public API exports are discoverable via IDE auto-completion and documented in module headers
- No internal implementation details (private helpers, internal state) are exposed through public exports

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler type checking during build, ESLint rules enforcing named exports, code review checklists, and automated tests importing public APIs are mandatory verification gates. Violations result in build failures, linting warnings, or code review feedback requiring remediation.
</enforcement>