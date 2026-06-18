# Standardize Public API Contracts Through Exported Interfaces and Types: Modules That Provide

These rules are ALWAYS ACTIVE for all modules and libraries that expose public APIs within the codebase, including agent implementations, core utilities, CLI handlers, configuration loaders, and top-level modules.

### Rules

- **R-API-001** MUST: All modules that provide functionality to other modules MUST export explicit public API contracts through TypeScript interfaces, types, or classes.
- **R-API-002** MUST: Follow the established pattern of prefixing interface names with 'I' (IAgent, IAgentConfig) for interface definitions that define contracts, while using descriptive names for type aliases (UnifiedLoadOptions).
- **R-API-003** MUST: Separate interface definitions into dedicated files (e.g., IAgent.ts) when the interface is implemented by multiple classes or consumed by multiple modules.
- **R-API-004** MUST: Export public functions with explicit return type annotations, especially for async operations (loadUnifiedConfig, discoverSkills, applyRulerConfig) to ensure contract clarity.
- **R-API-005** SHOULD: Co-locate related interfaces and types with their primary implementation when they form a cohesive module boundary.

### Verify

```bash
# Count exported interfaces
grep -r "export.*interface" src/ --include="*.ts" | wc -l

# Count exported agent classes
grep -r "export.*class.*Agent" src/agents/ --include="*.ts" | wc -l

# Verify TypeScript strict mode compilation
tsc --noEmit --strict && echo "Type checking passed"
```

**Accept when:**
- All modules in src/agents/ and src/core/ that are imported by other modules export at least one public interface, type, or class definition.
- TypeScript compilation succeeds with strict mode enabled, confirming all exported contracts are properly typed.
- Code review confirms that no internal implementation details are exposed through public exports, maintaining clear API boundaries.

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compiler strict mode checks MUST pass during CI build process. Code review MUST verify public API contracts are defined for new modules. Architecture review is REQUIRED for any changes to widely-used interfaces like IAgent or IAgentConfig.
</enforcement>