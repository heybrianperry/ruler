# Standardize Public API Contract Exports via Named Class/Interface Patterns: Agent Implementations Export

These rules are ALWAYS ACTIVE for all TypeScript modules in src/agents/, src/core/, src/paths/, and src/vscode/ that export public API contracts, configuration utilities, or reusable functionality to other parts of the codebase.

### Rules

- **R-AGENT-EXPORT-001** MUST: Agent implementations MUST export a named class (e.g., FirebenderAgent, MistralVibeAgent, GeminiCliAgent) that implements the IAgent interface contract.
- **R-AGENT-EXPORT-002** MUST: Configuration and utility modules MUST export named interfaces and type definitions alongside implementation functions (e.g., loadConfig with LoadedConfig, ConfigOptions).
- **R-AGENT-EXPORT-003** MUST: Use TypeScript's 'export { ... }' syntax at the end of files to create a clear manifest of public exports.
- **R-AGENT-EXPORT-004** SHOULD: Document public interfaces with JSDoc comments explaining their purpose, usage constraints, and behavioral guarantees.
- **R-AGENT-EXPORT-005** MAY: Use default exports only in documented exception cases (single utility functions with no associated types or configuration).

### Verify

```bash
# Count named exports across public API modules
grep -r 'export class\|export interface\|export type' src/agents/ src/core/ src/paths/ src/vscode/ | wc -l

# Verify minimal default exports in public modules
grep -r 'export default' src/agents/ src/core/ | grep -v '.test.ts' | wc -l

# Count agent implementations exporting named classes
npx ts-node -e "import * as agents from './src/agents'; console.log(Object.keys(agents).filter(k => k.includes('Agent')).length)"
```

**Accept when:**
- All agent modules export named classes implementing IAgent interface
- Configuration and utility modules export named interfaces/types alongside implementation functions
- Default exports are used only in documented exception cases (single utility functions)
- Public API exports are discoverable via IDE auto-completion and documented in module headers
- TypeScript compiler successfully type-checks all public API contracts during build

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for modules in scope. Violations result in build failures for TypeScript interface contract violations and linting warnings for default exports in public API modules.
</enforcement>