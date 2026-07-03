# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Hash Functions Exposed

These rules are ALWAYS ACTIVE for all cryptographic hashing operations, data integrity verification, content addressing, cache key generation, and any code importing or using hash functions from the codebase.

### Rules

- **R-HASH-001** MUST: Hash functions MUST be exposed through public API contracts (sha256, stableJson) for consistent usage across the codebase.
- **R-HASH-002** MUST: All data integrity verification operations MUST use the centralized hash API from src/core/hash.ts rather than implementing custom hashing logic.
- **R-HASH-003** MUST: All objects requiring hashing MUST apply stableJson serialization before passing to sha256 to ensure deterministic output.
- **R-HASH-004** MUST: Direct usage of crypto.createHash outside of src/core/hash.ts is prohibited; all hashing MUST go through the centralized API.
- **R-HASH-005** SHOULD: Consider implementing hash result caching for immutable objects that are hashed frequently to amortize serialization costs.
- **R-HASH-006** SHOULD: Profile critical paths and implement caching strategies for frequently hashed values to mitigate performance degradation in high-throughput scenarios.

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
- Content addressing and cache key generation use the exposed hash functions

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for cryptographic hashing operations.
</enforcement>