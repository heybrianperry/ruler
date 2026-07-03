# Standardize SHA-256 Hashing via Node.js Crypto Module for Deterministic Content Fingerprinting: Hash Input Data

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files in the codebase that perform content hashing, fingerprinting, or cryptographic operations.

### Rules

- **R-HASH-001** MUST: Hash input data MUST be normalized through stableJson serialization to ensure deterministic output across equivalent object representations.
- **R-HASH-002** MUST: All content hashing operations MUST use sha256 and stableJson exported from src/core/hash.ts rather than directly invoking crypto.createHash.
- **R-HASH-003** SHOULD: For large content payloads, consider streaming hash updates using hash.update() in chunks rather than loading entire content into memory before hashing.
- **R-HASH-004** SHOULD: When persisting hash values, prefix with algorithm identifier (e.g., 'sha256:') to enable future algorithm migration without breaking existing stored hashes.
- **R-HASH-005** MUST: Document expected input types for stableJson normalization and handle serialization errors gracefully with fallback strategies or clear error messages.

### Verify

```bash
# Verify no direct crypto.createHash('sha256') invocations exist outside src/core/hash.ts
grep -r "createHash('sha256')" src/ | grep -v src/core/hash.ts | wc -l | grep -q '^0$'

# Verify all modules requiring content hashing import from src/core/hash.ts
grep -r "from ['\"].*hash['\"]" src/ | grep -E "(sha256|stableJson)" | wc -l

# Verify stableJson produces identical serialization for equivalent objects
node -e "const {sha256, stableJson} = require('./src/core/hash'); console.assert(sha256(stableJson({b:2,a:1})) === sha256(stableJson({a:1,b:2})))"
```

**Accept when:**
- No direct crypto.createHash('sha256') invocations exist outside src/core/hash.ts
- All modules requiring content hashing import sha256 and/or stableJson from src/core/hash.ts
- stableJson produces identical serialization for equivalent objects regardless of property order
- All content fingerprinting operations use the centralized src/core/hash.ts API
- Serialization errors are handled gracefully with clear error messages

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via grep must pass, unit tests validating stableJson deterministic serialization must pass, and code review must confirm all content hashing bypasses src/core/hash.ts API only with documented exceptions approved by security and architecture review board.
</enforcement>