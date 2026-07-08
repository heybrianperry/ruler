# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Hash Functions Exposed

These rules are ALWAYS ACTIVE for all cryptographic hash operations, data integrity verification, content addressing workflows, and any code requiring deterministic hash values across runtime environments.

### Rules

- **R-HASH-001** MUST: Hash functions MUST be exposed through public API contracts (sha256, stableJson) for consistent usage across the codebase.
- **R-HASH-002** MUST: All cryptographic hash operations MUST use the centralized sha256 function from src/core/hash.ts.
- **R-HASH-003** MUST: All object hashing operations MUST use stableJson serialization before passing to sha256() to ensure deterministic results.
- **R-HASH-004** MUST: No direct crypto.createHash calls are permitted outside the core hash module (src/core/hash.ts).
- **R-HASH-005** SHOULD: Consider implementing hash result caching for immutable objects that are hashed frequently to reduce computational overhead.
- **R-HASH-006** SHOULD: Document the specific use case when adding new hash operations to ensure SHA-256 is appropriate (not password hashing, not performance-critical non-security paths).

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
- Hash utilities are imported from src/core/hash.ts using public API contracts

<enforcement>
Claude Code MUST NOT skip or defer verification. All hash operations MUST comply with R-HASH-001 through R-HASH-006. CI pipeline MUST fail if direct crypto.createHash usage is detected outside approved modules. Code review MUST block merges that bypass standardized hash API.
</enforcement>