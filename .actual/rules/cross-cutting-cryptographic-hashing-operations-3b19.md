# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Cryptographic Hashing Operations

These rules are ALWAYS ACTIVE for all cryptographic hashing operations, data integrity verification, content addressing, cache key generation, and cryptographic fingerprinting of configuration or state throughout the codebase.

### Rules

- **R-CRYPTO-001** MUST: All cryptographic hashing operations MUST use SHA-256 algorithm via Node.js crypto.createHash('sha256')
- **R-CRYPTO-002** MUST: Import sha256 and stableJson from src/core/hash.ts rather than implementing custom hashing
- **R-CRYPTO-003** MUST: For objects, always serialize with stableJson before passing to sha256 to ensure determinism
- **R-CRYPTO-004** SHOULD: Consider caching hash results for immutable objects to avoid redundant computation
- **R-CRYPTO-005** SHOULD: Document hash algorithm version in persistent storage schemas to support future migration

### Verify

```bash
# Verify no direct crypto.createHash('sha256') calls exist outside src/core/hash.ts
grep -r "createHash('sha256')" src/ | grep -v src/core/hash.ts

# Verify no direct crypto module requires outside src/core/hash.ts (excluding tests)
grep -r "require('crypto')" src/ | grep -v src/core/hash.ts | grep -v test

# Verify stable JSON serialization produces identical hashes for equivalent objects
node -e "const {sha256, stableJson} = require('./src/core/hash'); console.log(sha256(stableJson({b:2,a:1})) === sha256(stableJson({a:1,b:2})))"
```

**Accept when:**
- No direct crypto.createHash('sha256') calls exist outside src/core/hash.ts
- All modules requiring hashing import from src/core/hash.ts
- Stable JSON serialization produces identical hashes for equivalent objects regardless of key order
- All data integrity verification, content addressing, cache key generation, and cryptographic fingerprinting operations use the core hash module API

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis scanning for direct crypto module usage outside the core hash module is mandatory. Code review checklist requiring hash module imports is mandatory. Unit tests verifying hash determinism across object key orderings are mandatory.
</enforcement>