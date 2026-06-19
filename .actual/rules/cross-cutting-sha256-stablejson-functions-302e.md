# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Sha256 Stablejson Functions

These rules are ALWAYS ACTIVE for all data integrity verification operations, content addressing, cache key generation, and cryptographic fingerprinting of configuration or state.

### Rules

- **R-SHA256-001** MUST: The sha256 and stableJson functions MUST be exposed as public API contracts in the core library.
- **R-SHA256-002** MUST: All data integrity verification operations MUST use sha256 and stableJson from src/core/hash.ts rather than implementing custom hashing.
- **R-SHA256-003** MUST: For objects, always serialize with stableJson before passing to sha256 to ensure determinism.
- **R-SHA256-004** MUST: No direct crypto.createHash('sha256') calls shall exist outside src/core/hash.ts.
- **R-SHA256-005** MUST: All modules requiring hashing MUST import from src/core/hash.ts.
- **R-SHA256-006** SHOULD: Consider caching hash results for immutable objects to avoid redundant computation.
- **R-SHA256-007** SHOULD: Document hash algorithm version in persistent storage schemas to support future migration.

### Verify

```bash
# Verify no direct SHA-256 usage outside core hash module
grep -r "createHash('sha256')" src/ | grep -v src/core/hash.ts

# Verify no direct crypto module imports outside core hash module
grep -r "require('crypto')" src/ | grep -v src/core/hash.ts | grep -v test

# Verify stable JSON serialization produces identical hashes for equivalent objects
node -e "const {sha256, stableJson} = require('./src/core/hash'); console.log(sha256(stableJson({b:2,a:1})) === sha256(stableJson({a:1,b:2})))"
```

**Accept when:**
- No direct crypto.createHash('sha256') calls exist outside src/core/hash.ts
- All modules requiring hashing import from src/core/hash.ts
- Stable JSON serialization produces identical hashes for equivalent objects regardless of key order
- All hash operations use the public API contracts from core library

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis scanning for direct crypto module usage outside core hash module is mandatory. Code review checklist requiring hash module imports is mandatory. Unit tests verifying hash determinism across object key orderings are mandatory. CI pipeline MUST fail on detection of crypto usage outside core module.
</enforcement>