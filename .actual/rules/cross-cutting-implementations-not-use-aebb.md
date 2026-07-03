# Standardize SHA-256 Hashing via Node.js Crypto Module for Deterministic Content Fingerprinting: Implementations Not Use

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript implementations requiring content fingerprinting, deduplication, or content-addressable storage patterns.

### Rules

- **R-HASH-001** MUST_NOT: Implementations MUST NOT use alternative hash algorithms (MD5, SHA-1) for content fingerprinting due to known collision vulnerabilities.
- **R-HASH-002** MUST: All content hashing operations MUST import and use `sha256` and/or `stableJson` from `src/core/hash.ts` rather than directly invoking `crypto.createHash`.
- **R-HASH-003** MUST: Direct `crypto.createHash('sha256')` invocations MUST NOT appear outside `src/core/hash.ts`.
- **R-HASH-004** SHOULD: For large content payloads, consider streaming hash updates using `hash.update()` in chunks rather than loading entire content into memory before hashing.
- **R-HASH-005** SHOULD: When persisting hash values, prefix with algorithm identifier (e.g., 'sha256:') to enable future algorithm migration without breaking existing stored hashes.

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
- No direct `crypto.createHash('sha256')` invocations exist outside `src/core/hash.ts`
- All modules requiring content hashing import `sha256` and/or `stableJson` from `src/core/hash.ts`
- `stableJson` produces identical serialization for equivalent objects regardless of property order
- Code review checklist confirms use of `src/core/hash.ts` API for all content fingerprinting operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting implementations. Violations detected by CI pipeline or static analysis MUST block merge unless approved exception is documented with security and architecture review board sign-off.
</enforcement>