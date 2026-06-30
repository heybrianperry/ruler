# Standardize SHA-256 Hashing via Node.js Crypto Module for Deterministic Content Fingerprinting: Sha256 Stablejson Functions

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files in the codebase that perform content hashing, fingerprinting, or cryptographic operations.

### Rules

- **R-SHA256-001** MUST: The sha256 and stableJson functions MUST be exposed as public API contracts from src/core/hash.ts
- **R-SHA256-002** MUST: All modules requiring content hashing MUST import sha256 and/or stableJson from src/core/hash.ts rather than directly invoking crypto.createHash
- **R-SHA256-003** MUST: No direct crypto.createHash('sha256') invocations are permitted outside src/core/hash.ts
- **R-SHA256-004** SHOULD: For large content payloads, consider streaming hash updates using hash.update() in chunks rather than loading entire content into memory before hashing
- **R-SHA256-005** SHOULD: When persisting hash values, prefix with algorithm identifier (e.g., 'sha256:') to enable future algorithm migration without breaking existing stored hashes
- **R-SHA256-006** SHOULD: Document expected input types for stableJson normalization and handle serialization errors gracefully with fallback strategies or clear error messages

### Verify

```bash
# Verify no direct crypto.createHash('sha256') invocations exist outside src/core/hash.ts
grep -r "createHash('sha256')" src/ | grep -v src/core/hash.ts | wc -l | grep -q '^0$'

# Verify all modules requiring content hashing import from src/core/hash.ts
grep -r "from ['\"].*hash['\"]" src/ | grep -E "(sha256|stableJson)" | wc -l

# Verify stableJson produces deterministic serialization regardless of property order
node -e "const {sha256, stableJson} = require('./src/core/hash'); console.assert(sha256(stableJson({b:2,a:1})) === sha256(stableJson({a:1,b:2})))"
```

**Accept when:**
- No direct crypto.createHash('sha256') invocations exist outside src/core/hash.ts
- All modules requiring content hashing import sha256 and/or stableJson from src/core/hash.ts
- stableJson produces identical serialization for equivalent objects regardless of property order
- Code review checklist confirms use of src/core/hash.ts API for all content fingerprinting operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes that introduce or modify hashing logic. Violations detected by CI pipeline or static analysis MUST be remediated or approved through documented exception process before merge.
</enforcement>