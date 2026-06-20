# Standardize Public API Contracts Through Exported Interfaces and Types: Configuration Loaders Utility

These rules are ALWAYS ACTIVE for all modules and libraries that expose public APIs within the codebase, including agent implementations, core utilities, CLI handlers, configuration loaders, and top-level exported modules.

### Rules

- **R-API-001** MUST: Configuration loaders and utility modules MUST export typed interfaces for options and return values (e.g., UnifiedLoadOptions, IAgentConfig, IAgent).
- **R-API-002** MUST: All agent implementations in src/agents/ MUST export their class definitions as public contracts with explicit type annotations.
- **R-API-003** MUST: Core utility modules in src/core/ that are imported by other modules MUST export typed interfaces separating internal logic from external consumption.
- **R-API-004** MUST: Public functions exported from modules MUST include explicit return type annotations, especially for async operations.
- **R-API-005** SHOULD: Interface names SHOULD be prefixed with 'I' (IAgent, IAgentConfig) for interface definitions that define contracts.
- **R-API-006** SHOULD: Separate interface definitions into dedicated files (e.g., IAgent.ts) when the interface is implemented by multiple classes or consumed by multiple modules.
- **R-API-007** SHOULD: Related interfaces and types SHOULD be co-located with their primary implementation when they form a cohesive module boundary.
- **R-API-008** MAY: Internal helper functions not exported from modules MAY omit explicit interface definitions.
- **R-API-009** MAY: Test utilities and fixtures MAY be excluded from public API contract requirements.

### Verify

```bash
# Count exported interfaces across the codebase
grep -r "export.*interface" src/ --include="*.ts" | wc -l

# Count exported agent classes
grep -r "export.*class.*Agent" src/agents/ --include="*.ts" | wc -l

# Verify TypeScript strict mode compilation
tsc --noEmit --strict && echo "Type checking passed"

# Check for exported members without type annotations
grep -r "export.*function" src/ --include="*.ts" | grep -v ": " | head -20
```

**Accept when:**
- All modules in src/agents/ and src/core/ that are imported by other modules export at least one public interface, type, or class definition.
- TypeScript compilation succeeds with strict mode enabled, confirming all exported contracts are properly typed.
- Code review confirms that no internal implementation details are exposed through public exports, maintaining clear API boundaries.
- All exported functions include explicit return type annotations.
- Interface naming conventions (I-prefix for interfaces, descriptive names for type aliases) are consistently applied.

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler strict mode checks MUST pass during CI build process. Code review MUST verify public API contracts are defined for new modules. Architecture review is REQUIRED for any changes to widely-used interfaces like IAgent or IAgentConfig.
</enforcement>