<rule_activation id="ac56d9cd-a4d8-4e00-aaf8-51dc8c8dfb64" title="Standardize Public API Export Patterns for Agent Integration: Core Utility Modules" applies_to="**/*">
These rules are ALWAYS ACTIVE for all agent implementations and public API modules. All new agent integrations and external API surfaces MUST comply with these export patterns.
</rule_activation>

### Rules

- **R-API-001** MUST: Core utility modules MUST export pure functions with well-defined input/output contracts and no side effects.
- **R-API-002** MUST: All agent implementation files in src/agents/ MUST use named exports (no default exports, except during migration per EXC-001).
- **R-API-003** MUST: All agent classes MUST follow the naming convention with 'Agent' suffix.
- **R-API-004** MUST: Central index file (src/agents/index.ts) MUST contain exports for all agent implementations.
- **R-API-005** MUST: Integration modules in src/mcp/ and similar directories MUST follow standardized export patterns.
- **R-API-006** SHOULD: Use TypeScript's 'export type' and 'export interface' to clearly separate type exports from value exports.
- **R-API-007** SHOULD: Consider using barrel exports (index.ts files) at the agent directory level to simplify imports for consumers.

### Verify

```bash
# Verify no default exports in agent implementation files (excluding test files)
grep -r 'export default' src/agents/*.ts | grep -v '.test.ts' | wc -l | grep -q '^0$'

# Verify all agent classes follow naming convention
grep -r 'export class.*Agent' src/agents/*.ts | wc -l

# Verify central index file exists and contains agent exports
test -f src/agents/index.ts && grep -q 'export.*Agent' src/agents/index.ts
```

**Accept when:**
- No default exports found in agent implementation files (excluding test files)
- All agent classes follow the naming convention with 'Agent' suffix
- Central index file exists and contains exports for all agent implementations
- All core utility modules export pure functions with well-defined contracts
- Integration modules follow standardized export patterns

<enforcement>
Claude Code MUST NOT skip or defer verification. All verification commands MUST pass before accepting changes to agent implementations or public API modules. CI pipeline verification and code review enforcement are mandatory.
</enforcement>