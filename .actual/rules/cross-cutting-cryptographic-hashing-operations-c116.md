# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Cryptographic Hashing Operations

These rules are ALWAYS ACTIVE for all cryptographic hashing operations, data integrity verification, content addressing, cache key generation, and API contracts requiring stable hash outputs.

### Rules

- **R-CRYPTO-001** MUST: All cryptographic hashing operations MUST use SHA-256 via the Node.js crypto module's createHash function.
- **R-CRYPTO-002** MUST: All hash operations MUST import hash utilities from src/core/hash.ts using the public API contracts (sha256, stableJson) rather than implementing custom hashing logic.
- **R-CRYPTO-003** MUST: For objects requiring hashing, always apply stableJson serialization before passing to sha256 to ensure deterministic output.
- **R-CRYPTO-004** SHOULD: Consider implementing hash result caching for immutable objects that are hashed frequently to amortize serialization costs.
- **R-CRYPTO-005** SHOULD: Document the expected input types and constraints for hash functions to prevent misuse with unsupported data structures.

### Verify

```bash
# Verify no direct crypto.createHash usage outside core infrastructure
grep -r "createHash('sha256')" src/ | grep -v src/core/hash.ts

# Verify no direct crypto module imports outside core infrastructure
grep -r "require('crypto')" src/ | grep -v src/core/hash.ts

# Run hash-related tests to verify deterministic output
npm test -- --grep 'hash|sha256|stableJson'
```

**Accept when:**
- No direct usage of crypto.createHash outside of src/core/hash.ts (all hashing goes through centralized API)
- All hash-related tests pass demonstrating deterministic output for identical inputs
- Public API exports sha256 and stableJson functions that are imported by consumers
- No direct require('crypto') statements exist outside src/core/hash.ts

<enforcement>
Claude Code MUST NOT skip or defer verification. All cryptographic hashing operations MUST route through the centralized src/core/hash.ts module. Direct crypto module usage outside approved infrastructure is a violation.
</enforcement>