# Standardize Public API Contracts Through Exported Interfaces and Types: Public Functions That

These rules are ALWAYS ACTIVE for all modules and libraries that expose public APIs within the codebase, including agent implementations, core utilities, CLI handlers, configuration loaders, and top-level exported modules.

### Rules

- **R-API-001** SHOULD: Public functions that perform asynchronous operations SHOULD be explicitly typed with their return types and parameter interfaces.

### Verify

```bash
# Count exported interfaces across the codebase
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