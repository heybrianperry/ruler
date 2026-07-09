# Standardize SHA-256 with Stable JSON Serialization for Cryptographic Hashing: Components Not Implement

These rules are ALWAYS ACTIVE for all cryptographic hashing operations, data integrity verification, content addressing workflows, and any code requiring deterministic hash values across runtime environments.

### Rules

- **R-HASH-001** MUST NOT: Components MUST NOT implement custom hashing logic outside the standardized API contracts defined in `src/core/hash.ts`.
- **R-HASH-002** MUST: All cryptographic hash operations MUST use the centralized `sha256()` function from `src/core/hash.ts`.
- **R-HASH-003** MUST: All object hashing operations MUST use `stableJson()` serialization before passing to `sha256()` to ensure deterministic results across runtime environments.
- **R-HASH-004** MUST NOT: Direct `crypto.createHash()` calls MUST NOT appear outside the core hash module (`src/core/hash.ts`).
- **R-HASH-005** SHOULD: Hash result caching SHOULD be implemented for immutable objects that are hashed frequently to reduce computational overhead.
- **R-HASH-006** SHOULD: New hash operations SHOULD be documented with their specific use case to ensure SHA-256 is appropriate for the context.

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
- All cryptographic hash operations use the centralized `sha256()` function from `src/core/hash.ts`
- No direct `crypto.createHash()` calls exist outside the core hash module
- All object hashing operations use `stableJson()` serialization before hashing
- Test suite validates hash consistency across different object property orderings
- Hash operations in security-sensitive code paths are reviewed and approved
- Exception requests (if any) are documented with ticket references and expiration dates

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All cryptographic hashing operations MUST conform to the centralized API contract. Violations in security-sensitive code paths MUST trigger security team notification.
</enforcement>