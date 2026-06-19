# Standardize Subagent Discovery and Contract Propagation via Exported Public APIs: Public Contracts Use

These rules are ALWAYS ACTIVE for all modules in `src/core/` that implement subagent discovery, parsing, validation, or propagation, and for integration points with Claude that consume subagent configurations.

### Rules

- **R-PUBCON-001** SHOULD: Public contracts SHOULD use TypeScript types (ParsedFrontmatter, CopilotToolMapping) to enforce compile-time validation of subagent metadata structures.
- **R-PUBCON-002** MUST: Export all public contracts (discoverSubagents, getSelectedSubagentTargets, loadSubagentFile, parseFrontmatter, validateFrontmatter, propagateSubagentsForClaude) from module index files to establish clear API boundaries.
- **R-PUBCON-003** MUST: Use TypeScript's 'export type' for metadata structures (ParsedFrontmatter, CopilotToolMapping) to ensure compile-time validation without runtime overhead.
- **R-PUBCON-004** SHOULD: Implement comprehensive unit tests for each public contract function, mocking filesystem operations and parser dependencies to enable isolated testing.
- **R-PUBCON-005** SHOULD: Document public contracts with JSDoc annotations including parameter types, return types, error conditions, and usage examples to support API consumers.
- **R-PUBCON-006** MAY: Consider introducing a SubagentContract interface or type to formalize the expected structure of discovered subagent configurations.

### Verify

```bash
# Verify all public contract functions are exported
grep -r "export.*discoverSubagents\|export.*loadSubagentFile\|export.*propagateSubagentsForClaude" src/core/

# Verify parser dependencies are isolated to SubagentsUtils
grep -r "import.*js-yaml\|import.*@iarna/toml" src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts

# Run unit tests with coverage
npm test -- --testPathPattern="SubagentsProcessor|SubagentsUtils" --coverage

# Verify TypeScript strict mode compilation
npx tsc --strict --noEmit
```

**Accept when:**
- All public contract functions (discoverSubagents, getSelectedSubagentTargets, loadSubagentFile, parseFrontmatter, validateFrontmatter, propagateSubagentsForClaude) are exported from SubagentsProcessor.ts and SubagentsUtils.ts
- TypeScript compilation succeeds with strict type checking enabled for ParsedFrontmatter and CopilotToolMapping types
- Unit tests achieve >80% coverage for public contract functions with mocked filesystem and parser dependencies
- JSDoc annotations are present on all exported public contract functions
- No direct filesystem coupling exists outside of designated public contracts

<enforcement>
Claude Code MUST NOT skip or defer verification. TypeScript compilation failures block CI pipeline until type contract violations are resolved. Code review MUST reject PRs that bypass public contracts or introduce direct filesystem coupling.
</enforcement>