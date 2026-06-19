# Standardize Subagent Discovery and Contract Propagation via Exported Public APIs: Subagent Propagation External

These rules are ALWAYS ACTIVE for all modules in `src/core/` that implement subagent discovery, parsing, validation, or propagation, and for all integration points with Claude that consume subagent configurations.

### Rules

- **R-SUBAGENT-PROP-001** MUST: Subagent propagation to external systems MUST use atomic write operations (writeAgentsDirectoryAtomic) and expose propagateSubagentsForClaude as the public integration contract.

### Verify

```bash
# Verify all public contract functions are exported
grep -r "export.*discoverSubagents\|export.*loadSubagentFile\|export.*propagateSubagentsForClaude" src/core/

# Verify parser dependencies are isolated to SubagentsUtils and SubagentsProcessor
grep -r "import.*js-yaml\|import.*@iarna/toml" src/core/SubagentsUtils.ts src/core/SubagentsProcessor.ts

# Run unit tests with coverage
npm test -- --testPathPattern="SubagentsProcessor|SubagentsUtils" --coverage
```

**Accept when:**
- All public contract functions (discoverSubagents, getSelectedSubagentTargets, loadSubagentFile, parseFrontmatter, validateFrontmatter, propagateSubagentsForClaude) are exported from SubagentsProcessor.ts and SubagentsUtils.ts
- TypeScript compilation succeeds with strict type checking enabled for ParsedFrontmatter and CopilotToolMapping types
- Unit tests achieve >80% coverage for public contract functions with mocked filesystem and parser dependencies
- All subagent propagation operations use atomic write operations and the standardized propagateSubagentsForClaude contract

<enforcement>
Clause Code MUST NOT skip or defer verification. TypeScript compilation failures block CI pipeline. Code review rejects PRs that bypass public contracts or introduce direct filesystem coupling. Integration test failures trigger rollback of subagent configuration changes.
</enforcement>