# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Hash Utilities Centralized

These rules are ALWAYS ACTIVE for all cryptographic hashing operations, data integrity verification, content addressing workflows, and any code requiring deterministic hash values across runtime environments.

### Rules

- **R-HASH-001** MUST: All cryptographic hash operations MUST use the centralized sha256 function from src/core/hash.ts to prevent implementation drift and ensure consistent behavior.
- **R-HASH-002** MUST: No direct crypto.createHash calls are permitted outside src/core/hash.ts; all hashing MUST route through the core hash module's public API.
- **R-HASH-003** MUST: All object hashing operations MUST use stableJson serialization before passing to sha256() to ensure deterministic results across different runtime environments.
- **R-HASH-004** SHOULD: Hash utilities SHOULD be centralized in core infrastructure modules (src/core/hash.ts) to prevent implementation drift and reduce code duplication.
- **R-HASH-005** SHOULD: Frequently hashed immutable objects SHOULD implement hash result caching to reduce computational overhead in performance-sensitive contexts.
- **R-HASH-006** MAY: Non-cryptographic hash functions (e.g., xxHash, MurmurHash) MAY be used only for performance-critical paths where cryptographic properties are not required and collision risk is acceptable, with security team approval.

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
- Hash operations include documentation of their specific use case confirming SHA-256 is appropriate

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline validation.
</enforcement>