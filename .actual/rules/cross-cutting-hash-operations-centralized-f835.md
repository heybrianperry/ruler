# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Hash Operations Centralized

These rules are ALWAYS ACTIVE for all files in the codebase that perform cryptographic hashing, data integrity verification, content addressing, cache key generation, or cryptographic fingerprinting operations.

### Rules

- **R-HASH-001** MUST: Hash operations MUST be centralized in `src/core/hash.ts` to maintain consistency across all data integrity verification, content addressing, cache key generation, and cryptographic fingerprinting operations.
- **R-HASH-002** MUST: All modules requiring hashing MUST import `sha256` and `stableJson` from `src/core/hash.ts` rather than implementing custom hashing or calling `crypto.createHash()` directly.
- **R-HASH-003** MUST: For objects, always serialize with `stableJson` before passing to `sha256` to ensure deterministic hash output regardless of object key ordering.
- **R-HASH-004** SHOULD: Hash results for immutable objects SHOULD be cached to avoid redundant computation in performance-critical paths.
- **R-HASH-005** MUST: Direct calls to `crypto.createHash('sha256')` MUST NOT appear outside `src/core/hash.ts` except in test files.
- **R-HASH-006** MUST: Password hashing, HMAC operations, digital signatures, and random number generation MUST NOT use the centralized hash module; use bcrypt, argon2, or asymmetric cryptography libraries instead.
- **R-HASH-EX-001** MAY: Legacy systems requiring MD5 or SHA-1 for backward compatibility MAY request an exception through the security team with documented justification.

### Verify

```bash
# Check for direct crypto.createHash('sha256') calls outside core hash module
grep -r "createHash('sha256')" src/ | grep -v src/core/hash.ts

# Check for direct crypto module requires outside core hash module and tests
grep -r "require('crypto')" src/ | grep -v src/core/hash.ts | grep -v test

# Verify stable JSON serialization produces identical hashes for equivalent objects
node -e "const {sha256, stableJson} = require('./src/core/hash'); console.log(sha256(stableJson({b:2,a:1})) === sha256(stableJson({a:1,b:2})))"
```

**Accept when:**
- No direct `crypto.createHash('sha256')` calls exist outside `src/core/hash.ts`
- All modules requiring hashing import from `src/core/hash.ts`
- Stable JSON serialization produces identical hashes for equivalent objects regardless of key order
- All hash operations are verified to use the centralized API

<enforcement>
Claude Code MUST NOT skip or defer verification. All hash operations must be validated against these rules before code is approved.
</enforcement>