# Standardize Subagent Discovery and Contract Propagation via Exported Public APIs: Subagent Discovery Modules

These rules are ALWAYS ACTIVE for all modules in `src/core/` that implement subagent discovery, parsing, validation, or propagation functionality, and for integration points with Claude that consume subagent configurations.

### Rules

- **R-SUBAGENT-001** MUST: Subagent discovery modules MUST export public contracts for `discoverSubagents`, `getSelectedSubagentTargets`, and `getSubagentsGitignorePaths` to enable coordinated filesystem traversal and filtering.
- **R-SUBAGENT-002** MUST: All public contract functions (`discoverSubagents`, `getSelectedSubagentTargets`, `loadSubagentFile`, `parseFrontmatter`, `validateFrontmatter`, `propagateSubagentsForClaude`) MUST be exported from module index files to establish clear API boundaries.
- **R-SUBAGENT-003** MUST: TypeScript type definitions for subagent metadata (`ParsedFrontmatter`, `CopilotToolMapping`) MUST use `export type` to ensure compile-time validation without runtime overhead.
- **R-SUBAGENT-004** MUST: Public contracts MUST be documented with JSDoc annotations including parameter types, return types, error conditions, and usage examples.
- **R-SUBAGENT-005** SHOULD: Implement comprehensive unit tests for each public contract function, mocking filesystem operations and parser dependencies to enable isolated testing.
- **R-SUBAGENT-006** SHOULD: Consider introducing a `SubagentContract` interface or type to formalize the expected structure of discovered subagent configurations.

### Verify

```bash
# Verify all public contract functions are exported
grep -r "export.*discoverSubagents\|export.*loadSubagentFile\|export.*propagateSubagentsForClaude" src/core/

# Verify parser dependencies are properly imported
grep -r "import.*js-yaml\|import.*@iarna/toml" src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts

# Run unit tests with coverage
npm test -- --testPathPattern="SubagentsProcessor|SubagentsUtils" --coverage

# Verify TypeScript compilation with strict mode
npx tsc --strict --noEmit
```

**Accept when:**
- All public contract functions (`discoverSubagents`, `getSelectedSubagentTargets`, `loadSubagentFile`, `parseFrontmatter`, `validateFrontmatter`, `propagateSubagentsForClaude`) are exported from `SubagentsProcessor.ts` and `SubagentsUtils.ts`
- TypeScript compilation succeeds with strict type checking enabled for `ParsedFrontmatter` and `CopilotToolMapping` types
- Unit tests achieve >80% coverage for public contract functions with mocked filesystem and parser dependencies
- All public contract functions include JSDoc annotations with parameter types, return types, and usage examples
- No direct filesystem coupling exists outside of designated public contract functions

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failures block CI pipeline until type contract violations are resolved. Code review MUST reject PRs that bypass public contracts or introduce direct filesystem coupling.
</enforcement>