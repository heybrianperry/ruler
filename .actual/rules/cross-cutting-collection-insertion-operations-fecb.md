# Standardize Set-Based Collection Management for Deduplication in Subagent Processing: Collection Insertion Operations

These rules are ALWAYS ACTIVE for all subagent processing modules in src/core/ that manage collections of targets, aliases, or identifiers requiring deduplication, including functions using fs/promises, js-yaml, or @iarna/toml for configuration aggregation, and public API contracts returning collections.

### Rules

- **R-COLL-001** MUST: Collection insertion operations MUST use the .add() method pattern (e.g., 'targets.add(target)', 'mapped.add(alias)') to maintain Set semantics.
- **R-COLL-002** MUST: Initialize collections as Set instances at function entry (e.g., 'const targets = new Set<string>()').
- **R-COLL-003** MUST: Convert Set collections to arrays only at API boundaries using Array.from() or spread operator (e.g., 'return Array.from(targets)' or 'return [...targets]').
- **R-COLL-004** SHOULD: For complex objects, extract a unique primitive key and deduplicate on that key, then use Map to preserve object references (e.g., 'const uniqueMap = new Map(items.map(item => [item.id, item]))').
- **R-COLL-005** SHOULD: Add TypeScript type annotations to make Set usage explicit in function signatures (e.g., 'function collectTargets(): Set<string>' internally, 'function getTargets(): string[]' for public APIs).

### Verify

```bash
# Verify Set.add() pattern usage for collection variables
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
Claude Code MUST NOT skip or defer verification. All collection insertion operations in scope MUST use Set.add() pattern. Code review MUST verify Set usage before merge. CI pipeline MUST fail if grep verification commands do not find expected Set patterns in modified files.
</enforcement>