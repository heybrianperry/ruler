# Standardize Subagent Discovery and Contract Propagation via Exported Public APIs: Modules Expose Test

These rules are ALWAYS ACTIVE for all modules in `src/core/` that implement subagent discovery, parsing, validation, or propagation, and for integration points with Claude that consume subagent configurations.

### Rules

- **R-SUBAGENT-001** MAY: Modules MAY expose test-only reset functions (_resetExperimentalWarningForTests) to support isolated unit testing without global state pollution.

### Verify

```bash
# Verify public contract exports
grep -r "export.*discoverSubagents\|export.*loadSubagentFile\|export.*propagateSubagentsForClaude" src/core/

# Verify parser dependencies are isolated
grep -r "import.*js-yaml\|import.*@iarna/toml" src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts

# Run unit tests with coverage
npm test -- --testPathPattern="SubagentsProcessor|SubagentsUtils" --coverage
```

**Accept when:**
- All public contract functions (discoverSubagents, getSelectedSubagentTargets, loadSubagentFile, parseFrontmatter, validateFrontmatter, propagateSubagentsForClaude) are exported from SubagentsProcessor.ts and SubagentsUtils.ts
- TypeScript compilation succeeds with strict type checking enabled for ParsedFrontmatter and CopilotToolMapping types
- Unit tests achieve >80% coverage for public contract functions with mocked filesystem and parser dependencies
- Test-only reset functions (_resetExperimentalWarningForTests) are present in modules requiring isolated test state management

<enforcement>
Claude Code MUST NOT skip or defer verification of public contract exports and test-only function presence. TypeScript compilation failures block CI pipeline until type contract violations are resolved.
</enforcement>