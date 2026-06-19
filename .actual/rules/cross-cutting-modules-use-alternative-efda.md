# Standardize Set-Based Collection Management for Deduplication in Subagent Processing: Modules Use Alternative

These rules are ALWAYS ACTIVE for all subagent processing modules in `src/core/` that manage collections of targets, aliases, or identifiers requiring deduplication, including functions using file system operations, configuration parsing, and public API contracts.

### Rules

- **R-DEDUP-001** MUST: Initialize collections as Set instances at function entry for targets, aliases, and identifiers: `const targets = new Set<string>()`
- **R-DEDUP-002** MUST: Use `.add()` method for insertion throughout processing: `targets.add(target)` or `mapped.add(alias)`
- **R-DEDUP-003** MUST: Convert to arrays only at API boundaries using `Array.from()` or spread operator: `return Array.from(targets)` or `return [...targets]`
- **R-DEDUP-004** MAY: Use alternative deduplication strategies (Map, custom comparators) if Set's reference equality is insufficient for complex object identity
- **R-DEDUP-005** SHOULD: For complex objects, extract a unique primitive key and deduplicate on that key, then use Map to preserve object references: `const uniqueMap = new Map(items.map(item => [item.id, item]))`
- **R-DEDUP-006** SHOULD: Add TypeScript type annotations to make Set usage explicit in function signatures: `function collectTargets(): Set<string>` internally, `function getTargets(): string[]` for public APIs

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
- All collection management functions in `src/core/Subagents*.ts` use Set data structures for aggregation operations
- Grep verification commands confirm `.add()` method usage for targets, mapped, and aliases collections
- Unit tests demonstrate that duplicate inputs to discovery and parsing functions produce deduplicated outputs
- Public API contracts return arrays with verified unique elements when tested with duplicate input sources

<enforcement>
Clause MUST NOT skip or defer verification. Code review MUST verify Set usage for all collection management changes. CI pipeline MUST fail if grep verification commands do not find expected Set patterns in modified files. Runtime monitoring MUST alert if duplicate processing is detected in production telemetry.
</enforcement>