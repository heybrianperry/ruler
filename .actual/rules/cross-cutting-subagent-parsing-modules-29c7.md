# Standardize Subagent Discovery and Contract Propagation via Exported Public APIs: Subagent Parsing Modules

These rules are ALWAYS ACTIVE for all modules in `src/core/` that implement subagent discovery, parsing, validation, or propagation, and for integration points with Claude that consume subagent configurations.

### Rules

- **R-SUBAGENT-001** MUST: Subagent parsing modules MUST export `parseFrontmatter`, `validateFrontmatter`, and `loadSubagentFile` functions to enforce consistent metadata extraction from markdown files using js-yaml and @iarna/toml.
- **R-SUBAGENT-002** MUST: All public contract functions (discoverSubagents, getSelectedSubagentTargets, loadSubagentFile, parseFrontmatter, validateFrontmatter, propagateSubagentsForClaude) MUST be exported from SubagentsProcessor.ts and SubagentsUtils.ts to establish clear API boundaries.
- **R-SUBAGENT-003** MUST: TypeScript type definitions for subagent metadata (ParsedFrontmatter, CopilotToolMapping) MUST use 'export type' to ensure compile-time validation without runtime overhead.
- **R-SUBAGENT-004** SHOULD: Public contracts SHOULD be documented with JSDoc annotations including parameter types, return types, error conditions, and usage examples to support API consumers.
- **R-SUBAGENT-005** SHOULD: Comprehensive unit tests SHOULD be implemented for each public contract function, mocking filesystem operations and parser dependencies to enable isolated testing.

### Verify

```bash
# Verify all public contract functions are exported
grep -r "export.*discoverSubagents\|export.*loadSubagentFile\|export.*propagateSubagentsForClaude" src/core/

# Verify parser imports are present
grep -r "import.*js-yaml\|import.*@iarna/toml" src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts

# Run unit tests with coverage
npm test -- --testPathPattern="SubagentsProcessor|SubagentsUtils" --coverage

# Verify TypeScript compilation with strict mode
npx tsc --strict --noEmit
```

**Accept when:**
- All public contract functions (discoverSubagents, getSelectedSubagentTargets, loadSubagentFile, parseFrontmatter, validateFrontmatter, propagateSubagentsForClaude) are exported from SubagentsProcessor.ts and SubagentsUtils.ts
- TypeScript compilation succeeds with strict type checking enabled for ParsedFrontmatter and CopilotToolMapping types
- Unit tests achieve >80% coverage for public contract functions with mocked filesystem and parser dependencies
- All public contract functions include JSDoc annotations with parameter types, return types, and usage examples

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failures block CI pipeline until type contract violations are resolved. Code review MUST reject PRs that bypass public contracts or introduce direct filesystem coupling.
</enforcement>