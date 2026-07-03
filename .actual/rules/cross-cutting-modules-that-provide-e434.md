# Standardize Public API Contract Exports via Named Class/Interface Patterns: Modules That Provide

These rules are ALWAYS ACTIVE for all TypeScript modules in src/agents/, src/core/, src/paths/, and src/vscode/ that export public API contracts, as well as any new modules that provide reusable functionality to other parts of the codebase.

### Rules

- **R-API-001** MUST: All modules that provide public APIs MUST export named classes, interfaces, or type definitions as their primary contract mechanism.
- **R-API-002** MUST: Agent modules MUST export named classes that implement the IAgent interface alongside any agent-specific configuration types.
- **R-API-003** MUST: Configuration and utility modules MUST export both the loading/utility function and associated type definitions to provide complete contract information to consumers.
- **R-API-004** SHOULD: Use TypeScript's 'export { ... }' syntax at the end of files to create a clear manifest of public exports.
- **R-API-005** SHOULD: Document public interfaces with JSDoc comments explaining their purpose, usage constraints, and any important behavioral guarantees.
- **R-API-006** MAY: Default exports are permitted only for modules that provide a single, simple utility function with no associated types or configuration (EXC-001).

### Verify

```bash
# Count named exports across public API modules
grep -r 'export class\|export interface\|export type' src/agents/ src/core/ src/paths/ src/vscode/ | wc -l

# Verify minimal use of default exports in public API modules
grep -r 'export default' src/agents/ src/core/ | grep -v '.test.ts' | wc -l

# Verify agent exports are discoverable
npx ts-node -e "import * as agents from './src/agents'; console.log(Object.keys(agents).filter(k => k.includes('Agent')).length)"
```

**Accept when:**
- All agent modules export named classes implementing IAgent interface
- Configuration and utility modules export named interfaces/types alongside implementation functions
- Default exports are used only in documented exception cases (single utility functions)
- Public API exports are discoverable via IDE auto-completion and documented in module headers
- TypeScript compiler successfully type-checks all public API contracts during build

<enforcement>
Claude Code MUST NOT skip or defer verification. Build failures for modules that violate TypeScript interface contracts, linting warnings for default exports in public API modules, and code review feedback requiring interface definitions are mandatory enforcement points.
</enforcement>