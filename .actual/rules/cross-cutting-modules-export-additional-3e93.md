# Standardize Public API Contract Exports via Named Class/Interface Patterns: Modules Export Additional

These rules are ALWAYS ACTIVE for all TypeScript modules in src/agents/, src/core/, src/paths/, and src/vscode/ that export public API contracts, as well as any new modules that provide reusable functionality to other parts of the codebase.

### Rules

- **R-API-001** MAY: Modules MAY export additional utility types or helper interfaces to support consumers in correctly using the primary API contracts.

### Verify

```bash
# Count named exports across public API modules
grep -r 'export class\|export interface\|export type' src/agents/ src/core/ src/paths/ src/vscode/ | wc -l

# Verify minimal use of default exports in public modules
grep -r 'export default' src/agents/ src/core/ | grep -v '.test.ts' | wc -l

# Verify agent modules export named classes
npx ts-node -e "import * as agents from './src/agents'; console.log(Object.keys(agents).filter(k => k.includes('Agent')).length)"
```

**Accept when:**
- All agent modules export named classes implementing IAgent interface
- Configuration and utility modules export named interfaces/types alongside implementation functions
- Default exports are used only in documented exception cases (single utility functions)
- Public API exports are discoverable via IDE auto-completion and documented in module headers
- Modules provide clear, discoverable API boundaries with explicit contract definitions

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler type checking during build, ESLint rules enforcing named exports, code review checklists, and automated tests importing public APIs are mandatory verification gates.
</enforcement>