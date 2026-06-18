# Standardize Public API Contract Exports via Named Class/Interface Patterns: Public Functions Exported

These rules are ALWAYS ACTIVE for all TypeScript modules in src/agents/, src/core/, src/paths/, and src/vscode/ that export public API contracts, as well as any new modules that provide reusable functionality to other parts of the codebase.

### Rules

- **R-PUB-001** SHOULD: Public API functions SHOULD be exported as named exports rather than default exports to improve discoverability and refactoring safety.

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
- TypeScript compiler successfully type-checks all public API contracts during build

<enforcement>
Claude Code MUST NOT skip or defer verification. All public API modules MUST comply with R-PUB-001 unless an explicit exception (EXC-001) is documented and approved by the tech lead.
</enforcement>