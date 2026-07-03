# Standardize Set-Based Collection Management for Deduplication in Subagent Processing: Modules Exposing Public

These rules are ALWAYS ACTIVE for all subagent processing modules that manage collections of targets, aliases, or identifiers requiring deduplication, particularly in src/core/SubagentsProcessor.ts, src/core/SubagentsUtils.ts, and related modules exposing public API contracts.

### Rules

- **R-SETCOLL-001** SHOULD: Modules exposing public API contracts for collection retrieval SHOULD convert Set instances to arrays only at API boundaries to preserve internal deduplication guarantees.
- **R-SETCOLL-002** SHOULD: Initialize collections as Set instances at function entry using `const targets = new Set<string>()`.
- **R-SETCOLL-003** SHOULD: Use `.add()` for insertion throughout processing: `targets.add(target)` or `mapped.add(alias)`.
- **R-SETCOLL-004** SHOULD: Convert to arrays only at API boundaries using `Array.from()` or spread operator: `return Array.from(targets)` or `return [...targets]`.
- **R-SETCOLL-005** SHOULD: For complex objects, extract a unique primitive key and deduplicate on that key, then use Map to preserve object references: `const uniqueMap = new Map(items.map(item => [item.id, item]))`.
- **R-SETCOLL-006** SHOULD: Add TypeScript type annotations to make Set usage explicit in function signatures: `function collectTargets(): Set<string>` internally, `function getTargets(): string[]` for public APIs.

### Verify

```bash
# Verify Set.add() pattern usage for collection management
grep -r '\.add(' src/core/Subagents*.ts | grep -E '(targets|mapped|aliases)'

# Confirm Set initialization in collection management
grep -r 'new Set' src/core/Subagents*.ts

# Run tests verifying deduplication behavior
npm test -- --grep 'deduplication|unique'
```

**Accept when:**
- All collection management functions in src/core/Subagents*.ts use Set data structures for aggregation operations
- Grep verification commands confirm `.add()` method usage for targets, mapped, and aliases collections
- Unit tests demonstrate that duplicate inputs to discovery and parsing functions produce deduplicated outputs
- Public API contracts return arrays with verified unique elements when tested with duplicate input sources
- TypeScript type annotations explicitly mark Set usage in internal function signatures and array returns at API boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. All collection management changes in scope MUST pass grep pattern verification and unit test coverage before acceptance. Code review MUST verify Set usage patterns and API boundary conversions. Violations block merge until remediated or approved exception is documented.
</enforcement>