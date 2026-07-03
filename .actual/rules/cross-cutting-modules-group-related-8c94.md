# Standardize Public API Contract Exports via Named Class/Interface Patterns: Modules Group Related

These rules are ALWAYS ACTIVE for all TypeScript modules in src/agents/, src/core/, src/paths/, and src/vscode/ that export public API contracts, as well as any new modules that provide reusable functionality to other parts of the codebase.

### Rules

- **R-API-001** SHOULD: Modules SHOULD group related contracts (types, interfaces, classes) together and export them from a single module to establish clear API boundaries.
- **R-API-002** SHOULD: Named exports for classes and interfaces SHOULD be used to provide clear, discoverable API boundaries that support IDE tooling and human comprehension.
- **R-API-003** SHOULD: Public interfaces SHOULD be documented with JSDoc comments explaining their purpose, usage constraints, and any important behavioral guarantees.
- **R-API-004** SHOULD: TypeScript's 'export { ... }' syntax SHOULD be used at the end of files to create a clear manifest of public exports.
- **R-API-005** MAY: Namespace exports MAY be considered for grouping related utilities in future refactoring.
- **R-API-EXC-001** Exception: A module MAY provide a single, simple utility function with no associated types or configuration without explicit interface abstractions.

### Verify

```bash
# Count named exports across public API modules
grep -r 'export class\|export interface\|export type' src/agents/ src/core/ src/paths/ src/vscode/ | wc -l

# Verify minimal use of default exports in public modules
grep -r 'export default' src/agents/ src/core/ | grep -v '.test.ts' | wc -l

# Count agent classes exported with IAgent interface
npx ts-node -e "import * as agents from './src/agents'; console.log(Object.keys(agents).filter(k => k.includes('Agent')).length)"
```

**Accept when:**
- All agent modules export named classes implementing IAgent interface
- Configuration and utility modules export named interfaces/types alongside implementation functions
- Default exports are used only in documented exception cases (single utility functions)
- Public API exports are discoverable via IDE auto-completion and documented in module headers
- Named exports appear consistently across 14+ files with 90%+ confidence

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler type checking during build, ESLint rules enforcing named exports, code review checklists, and automated tests importing public APIs are mandatory verification gates. Build failures occur for modules violating TypeScript interface contracts. Linting warnings flag default exports in public API modules. Code review feedback requires interface definitions for multi-implementation scenarios.
</enforcement>