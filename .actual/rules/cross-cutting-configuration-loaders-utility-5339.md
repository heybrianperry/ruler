# Standardize Public API Contract Exports via Named Class/Interface Patterns: Configuration Loaders Utility

These rules are ALWAYS ACTIVE for all TypeScript modules in src/agents/, src/core/, src/paths/, and src/vscode/ that export public API contracts, configuration loaders, and utility functions.

### Rules

- **R-API-001** MUST: Configuration loaders and utility modules MUST export named interfaces or type definitions (e.g., UnifiedLoadOptions, VSCodeSettings, LoadedConfig) alongside their implementation functions.
- **R-API-002** MUST: All agent modules MUST export named classes implementing the IAgent interface as primary public contracts.
- **R-API-003** SHOULD: Use TypeScript's 'export { ... }' syntax at the end of files to create a clear manifest of public exports.
- **R-API-004** SHOULD: Document public interfaces with JSDoc comments explaining their purpose, usage constraints, and behavioral guarantees.
- **R-API-005** MAY: Use default exports only in documented exception cases (single utility functions with no associated types).

### Verify

```bash
# Count named exports across public API modules
grep -r 'export class\|export interface\|export type' src/agents/ src/core/ src/paths/ src/vscode/ | wc -l

# Verify minimal default exports in public modules
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
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for modules in scope. Violations must be caught during code review and build-time type checking.
</enforcement>