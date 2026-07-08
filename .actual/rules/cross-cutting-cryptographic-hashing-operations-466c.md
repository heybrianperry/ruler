# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Cryptographic Hashing Operations

These rules are ALWAYS ACTIVE for all cryptographic hashing operations, data integrity verification, content addressing workflows, and any code requiring deterministic hash values across runtime environments.

### Rules

- **R-CRYPTO-001** MUST: All cryptographic hashing operations MUST use SHA-256 algorithm via Node.js crypto.createHash('sha256') through the centralized API in src/core/hash.ts.
- **R-CRYPTO-002** MUST: All object hashing operations MUST use stableJson() serialization before passing to sha256() to ensure deterministic results across different runtime environments.
- **R-CRYPTO-003** MUST: No direct crypto.createHash calls are permitted outside the core hash module (src/core/hash.ts).
- **R-CRYPTO-004** SHOULD: Hash utilities SHOULD be imported from src/core/hash.ts using the public API contracts: sha256() for hashing and stableJson() for serialization.
- **R-CRYPTO-005** SHOULD: Frequently hashed immutable objects SHOULD implement hash result caching to reduce computational overhead.
- **R-CRYPTO-006** MAY: Non-cryptographic hash functions (xxHash, MurmurHash) MAY be used only for performance-critical paths where cryptographic properties are not required and collision risk is acceptable, with security team approval.

### Verify

```bash
# Verify no direct SHA-256 usage outside core module
grep -r "createHash('sha256')" src/ | grep -v src/core/hash.ts || echo 'No direct SHA-256 usage outside core module'

# Count imports of sha256 from core/hash
grep -r "import.*sha256.*from.*core/hash" src/ | wc -l

# Verify stable JSON tests exist
npm test -- --grep 'hash.*stable|stable.*json' 2>/dev/null || echo 'Verify stable JSON tests exist'
```

**Accept when:**
- All cryptographic hash operations use the centralized sha256 function from src/core/hash.ts
- No direct crypto.createHash calls exist outside the core hash module
- All object hashing operations use stableJson serialization before hashing
- Test suite validates hash consistency across different object property orderings
- Hash operations in security-sensitive code paths are documented with use case justification

<enforcement>
Claude Code MUST NOT skip or defer verification. All cryptographic hashing operations MUST comply with R-CRYPTO-001 through R-CRYPTO-006. Code review MUST block merges that bypass the standardized API. CI pipeline MUST fail if direct crypto.createHash usage is detected outside approved modules.
</enforcement>