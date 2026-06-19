# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Components Not Implement

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files in the codebase that perform cryptographic hashing, data integrity verification, content addressing, cache key generation, or cryptographic fingerprinting operations.

### Rules

- **R-HASH-001** MUST_NOT: Components MUST NOT implement custom hashing logic outside the core hash module (src/core/hash.ts).
- **R-HASH-002** MUST: All data integrity verification operations MUST use sha256 and stableJson from src/core/hash.ts.
- **R-HASH-003** MUST: All content addressing and deduplication operations MUST use sha256 and stableJson from src/core/hash.ts.
- **R-HASH-004** MUST: All cache key generation MUST use sha256 and stableJson from src/core/hash.ts.
- **R-HASH-005** MUST: All cryptographic fingerprinting of configuration or state MUST use sha256 and stableJson from src/core/hash.ts.
- **R-HASH-006** MUST: For objects, always serialize with stableJson before passing to sha256 to ensure determinism.
- **R-HASH-EX-001** Exception: Legacy systems MAY use MD5 or SHA-1 for backward compatibility only with documented justification and security team approval.

### Verify

```bash
# Verify no direct crypto.createHash('sha256') calls exist outside src/core/hash.ts
grep -r "createHash('sha256')" src/ | grep -v src/core/hash.ts

# Verify no direct crypto module requires outside src/core/hash.ts and tests
grep -r "require('crypto')" src/ | grep -v src/core/hash.ts | grep -v test

# Verify stable JSON serialization produces identical hashes for equivalent objects
node -e "const {sha256, stableJson} = require('./src/core/hash'); console.log(sha256(stableJson({b:2,a:1})) === sha256(stableJson({a:1,b:2})))"
```

**Accept when:**
- No direct crypto.createHash('sha256') calls exist outside src/core/hash.ts
- All modules requiring hashing import from src/core/hash.ts
- Stable JSON serialization produces identical hashes for equivalent objects regardless of key order
- All hash operations in scope use the public API from src/core/hash.ts
- Any exceptions are documented with security team approval and tracked in ADR addendum

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis scanning for direct crypto module usage outside core hash module is mandatory. Code review checklist requiring hash module imports is mandatory. Unit tests verifying hash determinism across object key orderings are mandatory. CI pipeline MUST fail on detection of crypto usage outside core module.
</enforcement>