# Standardize Set-Based Collection Management for Deduplication in Subagent Processing: Collection Management Operations

These rules are ALWAYS ACTIVE for all subagent processing modules in `src/core/` that manage collections of targets, aliases, or identifiers requiring deduplication, including functions using file system operations (fs/promises), configuration parsing (js-yaml, @iarna/toml), and public API contracts exposing collection management behavior.

### Rules

- **R-COLL-001** MUST: All collection management operations that aggregate targets, aliases, or identifiers from multiple sources MUST use Set data structures to enforce uniqueness at insertion time.
- **R-COLL-002** MUST: Initialize collections as Set instances at function entry using `const targets = new Set<string>()`.
- **R-COLL-003** MUST: Use `.add()` for insertion throughout processing: `targets.add(target)` or `mapped.add(alias)`.
- **R-COLL-004** MUST: Convert to arrays only at API boundaries using `Array.from()` or spread operator: `return Array.from(targets)` or `return [...targets]`.
- **R-COLL-005** MUST: Add TypeScript type annotations to make Set usage explicit in function signatures: `function collectTargets(): Set<string>` internally, `function getTargets(): string[]` for public APIs.
- **R-COLL-006** SHOULD: For complex objects requiring deduplication, extract a unique primitive key and use Map to preserve object references: `const uniqueMap = new Map(items.map(item => [item.id, item]))`.
- **R-COLL-007** MAY: Approve exceptions (EXC-001) when complex object identity requires custom equality semantics beyond reference equality, or (EXC-002) when performance profiling demonstrates Set overhead exceeds duplicate processing cost for specific high-frequency operations.

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
- All collection management functions in `src/core/Subagents*.ts` use Set data structures for aggregation operations.
- Grep verification commands confirm `.add()` method usage for targets, mapped, and aliases collections.
- Unit tests demonstrate that duplicate inputs to discovery and parsing functions produce deduplicated outputs.
- Public API contracts (discoverSubagents, getSelectedSubagentTargets, propagateSubagentsForClaude, loadSubagentFile) return arrays with verified unique elements when tested with duplicate input sources.
- TypeScript type annotations explicitly declare Set return types for internal functions and array return types for public APIs.

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if collection management uses arrays without documented exception approval. CI pipeline MUST fail if grep verification commands do not find expected Set patterns in modified files. Approved exceptions MUST be recorded in module JSDoc and architectural decision log.
</enforcement>