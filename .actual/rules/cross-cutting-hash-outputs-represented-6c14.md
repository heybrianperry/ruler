# Standardize SHA-256 Hashing via Node.js Crypto Module for Deterministic Content Fingerprinting: Hash Outputs Represented

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files in the `src/` directory that perform content hashing, fingerprinting, or cryptographic operations.

### Rules

- **R-HASH-001** MUST: Hash outputs MUST be represented as hexadecimal strings for human readability and consistent string-based comparisons.
- **R-HASH-002** MUST: All content hashing operations MUST use the `sha256` and `stableJson` functions exported from `src/core/hash.ts` rather than directly invoking `crypto.createHash('sha256')`.
- **R-HASH-003** SHOULD: For large content payloads, SHOULD use streaming hash updates via `hash.update()` in chunks rather than loading entire content into memory before hashing.
- **R-HASH-004** SHOULD: Hash values persisted to storage SHOULD be prefixed with algorithm identifier (e.g., `'sha256:'`) to enable future algorithm migration without breaking existing stored hashes.

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
- Hash outputs are consistently represented as hexadecimal strings in all consuming code

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to hashing logic or content fingerprinting operations.
</enforcement>