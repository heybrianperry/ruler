# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Hash Utility Functions

These rules are ALWAYS ACTIVE for all cryptographic hashing operations, data integrity verification, content addressing, cache key generation, and any code importing or using hash utility functions from core infrastructure.

### Rules

- **R-HASH-001** SHOULD: Hash utility functions SHOULD be centralized in core infrastructure modules (src/core/hash.ts) to prevent implementation drift and establish a single source of truth for cryptographic hashing operations.

### Verify

```bash
# Verify no direct usage of crypto.createHash outside src/core/hash.ts
grep -r "createHash('sha256')" src/ | grep -v src/core/hash.ts

# Verify no direct crypto module imports outside src/core/hash.ts
grep -r "require('crypto')" src/ | grep -v src/core/hash.ts

# Run hash-related tests to verify deterministic output
npm test -- --grep 'hash|sha256|stableJson'
```

**Accept when:**
- No direct usage of crypto.createHash outside of src/core/hash.ts (all hashing goes through centralized API)
- No direct require('crypto') statements outside of src/core/hash.ts
- All hash-related tests pass demonstrating deterministic output for identical inputs
- Public API exports sha256 and stableJson functions that are imported by consumers
- All data integrity verification operations use the centralized hash utility functions
- All content addressing and deduplication systems use the centralized hash utility functions
- All cache key generation requiring deterministic hashing uses the centralized hash utility functions

<enforcement>
Claude Code MUST NOT skip or defer verification. All pull requests introducing direct crypto usage outside src/core/hash.ts MUST be rejected. CI pipeline MUST fail if grep verification commands detect violations.
</enforcement>