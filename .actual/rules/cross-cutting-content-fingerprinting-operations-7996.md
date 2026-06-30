# Standardize SHA-256 Hashing via Node.js Crypto Module for Deterministic Content Fingerprinting: Content Fingerprinting Operations

These rules are ALWAYS ACTIVE for all files in the codebase that perform content fingerprinting, hashing, or data integrity verification operations.

### Rules

- **R-HASH-001** MUST: All content fingerprinting operations MUST use SHA-256 via Node.js crypto.createHash('sha256').
- **R-HASH-002** MUST: Import sha256 and stableJson from src/core/hash.ts rather than directly invoking crypto.createHash to maintain centralized control over hash implementation.
- **R-HASH-003** SHOULD: For large content payloads, use streaming hash updates via hash.update() in chunks rather than loading entire content into memory before hashing.
- **R-HASH-004** SHOULD: When persisting hash values, prefix with algorithm identifier (e.g., 'sha256:') to enable future algorithm migration without breaking existing stored hashes.
- **R-HASH-005** MUST: Document expected input types for stableJson normalization and handle serialization errors gracefully with fallback strategies or clear error messages.

### Verify

```bash
# Verify no direct crypto.createHash('sha256') invocations exist outside src/core/hash.ts
grep -r "createHash('sha256')" src/ | grep -v src/core/hash.ts | wc -l | grep -q '^0$'

# Verify all modules requiring content hashing import sha256 and/or stableJson from src/core/hash.ts
grep -r "from ['\"].*hash['\"]" src/ | grep -E "(sha256|stableJson)" | wc -l

# Verify stableJson produces identical serialization for equivalent objects regardless of property order
node -e "const {sha256, stableJson} = require('./src/core/hash'); console.assert(sha256(stableJson({b:2,a:1})) === sha256(stableJson({a:1,b:2})))"
```

**Accept when:**
- No direct crypto.createHash('sha256') invocations exist outside src/core/hash.ts
- All modules requiring content hashing import sha256 and/or stableJson from src/core/hash.ts
- stableJson produces identical serialization for equivalent objects regardless of property order
- All content fingerprinting operations use the centralized src/core/hash.ts API
- Hash operations include appropriate error handling for serialization failures

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to content fingerprinting operations. Code review MUST block merge if content hashing bypasses src/core/hash.ts API without documented exception.
</enforcement>