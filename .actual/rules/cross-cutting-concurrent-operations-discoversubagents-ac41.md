# Standardize Set-Based Collection Management for Deduplication in Subagent Processing: Concurrent Operations Discoversubagents

These rules are ALWAYS ACTIVE for all subagent processing modules that manage collections of targets, aliases, or identifiers requiring deduplication, including all modules in src/core/ managing subagent targets, functions using fs/promises, js-yaml, or @iarna/toml for configuration aggregation, and public API contracts returning collections (discoverSubagents, getSelectedSubagentTargets, loadSubagentFile).

### Rules

- **R-SBCM-001** SHOULD: Concurrent operations (discoverSubagents, listMarkdownFilesRecursive, propagateSubagentsForClaude) SHOULD use Set-based accumulation to prevent race-condition-induced duplicates.
- **R-SBCM-002** MUST: Initialize collections as Set instances at function entry using `const targets = new Set<string>()`.
- **R-SBCM-003** MUST: Use .add() for insertion throughout processing: `targets.add(target)` or `mapped.add(alias)`.
- **R-SBCM-004** MUST: Convert to arrays only at API boundaries using Array.from() or spread operator: `return Array.from(targets)` or `return [...targets]`.
- **R-SBCM-005** SHOULD: For complex objects, extract a unique primitive key and deduplicate on that key, then use Map to preserve object references: `const uniqueMap = new Map(items.map(item => [item.id, item]))`.
- **R-SBCM-006** MUST: Add TypeScript type annotations to make Set usage explicit in function signatures: `function collectTargets(): Set<string>` internally, `function getTargets(): string[]` for public APIs.

### Verify

```bash
# Verify Set.add() pattern usage for targets, mapped, and aliases collections
grep -r '\.add(' src/core/Subagents*.ts | grep -E '(targets|mapped|aliases)'

# Confirm Set initialization in collection management
grep -r 'new Set' src/core/Subagents*.ts

# Run tests verifying deduplication behavior
npm test -- --grep 'deduplication|unique'
```

**Accept when:**
- All collection management functions in src/core/Subagents*.ts use Set data structures for aggregation operations
- Grep verification commands confirm .add() method usage for targets, mapped, and aliases collections
- Unit tests demonstrate that duplicate inputs to discovery and parsing functions produce deduplicated outputs
- Public API contracts return arrays with verified unique elements when tested with duplicate input sources

<enforcement>
Claude Code MUST NOT skip or defer verification. All collection management changes in scope MUST pass grep verification commands and unit tests before acceptance. Code review MUST verify Set usage for all collection management changes. CI pipeline MUST fail if grep verification commands do not find expected Set patterns in modified files.
</enforcement>