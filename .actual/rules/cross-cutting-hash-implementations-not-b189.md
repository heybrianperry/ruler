# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Hash Implementations Not

These rules are ALWAYS ACTIVE for all cryptographic hashing operations, data integrity verification, content addressing, cache key generation, and any code that imports or uses the crypto module for hash operations.

### Rules

- **R-HASH-001** MUST NOT: Hash implementations MUST NOT use weaker algorithms (MD5, SHA-1) for cryptographic purposes.
- **R-HASH-002** MUST: All cryptographic hashing operations MUST use SHA-256 via the centralized API in src/core/hash.ts.
- **R-HASH-003** MUST: Direct usage of crypto.createHash() MUST NOT appear outside of src/core/hash.ts.
- **R-HASH-004** MUST: Objects requiring deterministic hashing MUST be serialized using stableJson before passing to sha256.
- **R-HASH-005** SHOULD: Implement hash result caching for immutable objects that are hashed frequently to amortize serialization costs.
- **R-HASH-006** SHOULD: Profile critical paths and implement caching strategies for frequently hashed values in high-throughput scenarios.

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
- No direct usage of crypto.createHash outside of src/core/hash.ts (all hashing goes through centralized API)
- No direct require('crypto') statements outside of src/core/hash.ts
- All hash-related tests pass demonstrating deterministic output for identical inputs
- Public API exports sha256 and stableJson functions that are imported by consumers
- Code review confirms all hash operations use the centralized API from src/core/hash.ts

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory and must be checked before accepting any code changes related to cryptographic hashing operations.
</enforcement>