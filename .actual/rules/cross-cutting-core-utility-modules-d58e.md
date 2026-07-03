# Standardize Public API Contract Exports in Agent and Core Modules: Core Utility Modules

These rules are ALWAYS ACTIVE for all TypeScript modules in `src/agents/` and `src/core/` directories that expose public interfaces for external consumption.

### Rules

- **R-API-001** MUST: Core utility modules MUST explicitly export all public types, interfaces, and functions intended for external consumption (e.g., ParsedFrontmatter, CopilotToolMapping, parseFrontmatter, validateFrontmatter).

### Verify

```bash
# Count explicit exports across agent and core modules
grep -r "^export " src/agents/ src/core/ | grep -v "export default" | wc -l

# Verify exports from OpenCodeAgent module
npx ts-node -e "import * as agents from './src/agents/OpenCodeAgent'; console.log(Object.keys(agents))"

# Find modules without any exports (excluding test files)
find src/agents src/core -name '*.ts' -exec grep -L "^export " {} \; | grep -v ".test.ts" | grep -v ".spec.ts"
```

**Accept when:**
- All agent modules export their primary class or interface with a descriptive name
- Core utility modules explicitly list all public exports without wildcard patterns
- No internal helper functions or private implementation details appear in module export lists
- TypeScript compilation succeeds with strict mode enabled, verifying all contract types are valid

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler checks during CI build process are mandatory. Code review checklist items for API surface area review must be completed. Automated linting rules checking export patterns must pass before merge approval.
</enforcement>