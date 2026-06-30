# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Object Structures Requiring

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-HASH-001** MUST: All object structures requiring hashing MUST be serialized using stable JSON serialization that produces deterministic output regardless of property insertion order.
- **R-HASH-002** MUST: Import hash utilities from `src/core/hash.ts` using the public API contracts (`sha256`, `stableJson`) rather than implementing custom hashing logic.
- **R-HASH-003** MUST: All hashing operations MUST use SHA-256 via the centralized `src/core/hash.ts` module; no direct usage of `crypto.createHash()` is permitted outside this module.
- **R-HASH-004** SHOULD: For objects requiring hashing, always apply `stableJson` serialization before passing to `sha256` to ensure deterministic output.
- **R-HASH-005** SHOULD: Consider implementing hash result caching for immutable objects that are hashed frequently to amortize serialization costs.

### Verify

```bash
# Check for direct SHA-256 usage outside core infrastructure
grep -r "createHash('sha256')" src/ | grep -v src/core/hash.ts

# Check for direct crypto module imports outside core infrastructure
grep -r "require('crypto')" src/ | grep -v src/core/hash.ts

# Run hash-related tests to verify deterministic output
npm test -- --grep 'hash|sha256|stableJson'
```

**Accept when:**
- No direct usage of `crypto.createHash` outside of `src/core/hash.ts` (all hashing goes through centralized API)
- All hash-related tests pass demonstrating deterministic output for identical inputs
- Public API exports `sha256` and `stableJson` functions that are imported by consumers
- Grep verification commands return no violations (exit code 0 or no matches)

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code affecting cryptographic hashing operations.
</enforcement>