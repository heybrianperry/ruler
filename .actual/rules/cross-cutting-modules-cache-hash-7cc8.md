# Standardize SHA-256 Hashing via Node.js Crypto Module for Deterministic Content Fingerprinting: Modules Cache Hash

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript modules requiring content fingerprinting, deduplication, or content-addressable storage patterns.

### Rules

- **R-HASH-001** MUST: Import `sha256` and `stableJson` from `src/core/hash.ts` rather than directly invoking `crypto.createHash('sha256')` to maintain centralized control over hash implementation.
- **R-HASH-002** MUST: Ensure no direct `crypto.createHash('sha256')` invocations exist outside `src/core/hash.ts`.
- **R-HASH-003** MAY: Modules MAY cache hash results for immutable content to avoid redundant computation.
- **R-HASH-004** SHOULD: For large content payloads, use streaming hash updates via `hash.update()` in chunks rather than loading entire content into memory before hashing.
- **R-HASH-005** SHOULD: Document expected input types for `stableJson` normalization and handle serialization errors gracefully with fallback strategies or clear error messages.
- **R-HASH-006** SHOULD: When persisting hash values, prefix with algorithm identifier (e.g., `'sha256:'`) to enable future algorithm migration without breaking existing stored hashes.

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
- All content fingerprinting operations use the centralized `src/core/hash.ts` API

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via grep/ripgrep, unit tests validating deterministic serialization, and code review checklists are mandatory. CI pipeline MUST fail if direct `crypto.createHash` invocations are detected outside `src/core/hash.ts`. Code review MUST block merge if content hashing bypasses the centralized API without documented exception.
</enforcement>