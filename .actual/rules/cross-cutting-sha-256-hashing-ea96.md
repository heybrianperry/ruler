# Standardize SHA-256 Hashing via Node.js Crypto Module for Data Integrity: Sha 256 Hashing

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files in the codebase that perform cryptographic hashing operations or handle data integrity verification.

### Rules

- **R-SHA256-001** MUST NOT: SHA-256 hashing MUST NOT be used for password storage or authentication credentials. Use bcrypt, argon2, or scrypt instead.
- **R-SHA256-002** MUST: All SHA-256 hashing operations MUST be performed via centralized utilities imported from `src/core/hash.ts` rather than direct calls to `crypto.createHash()`.
- **R-SHA256-003** MUST: When hashing complex objects, use the `stableJson` utility to serialize data structures in canonical form before hashing to ensure deterministic outputs.
- **R-SHA256-004** SHOULD: Document the purpose of each hash operation in code comments to clarify whether cryptographic properties are required for the specific use case.
- **R-SHA256-005** SHOULD: Consider caching hash results for immutable data structures to avoid redundant computation in performance-sensitive paths.
- **R-SHA256-006** MAY: Non-cryptographic hash functions (xxHash, MurmurHash) may be used for performance-critical internal use cases where adversarial input is not a concern (e.g., in-memory hash tables, non-security cache keys) with documented exception approval.

### Verify

```bash
# Detect direct crypto.createHash calls outside approved core modules
grep -r "createHash" --include="*.ts" --include="*.js" | grep -v "src/core/hash.ts" | grep -v "test" | grep -v "spec"

# Detect SHA-256 usage with non-standard algorithms
grep -r "crypto\.createHash" --include="*.ts" --include="*.js" | grep -v "'sha256'" | grep -v '"sha256"'

# Verify deterministic hashing tests exist
npm test -- --grep "hash.*deterministic" || echo 'Add tests for deterministic hashing behavior'
```

**Accept when:**
- All direct `crypto.createHash` calls outside `src/core/hash.ts` are migrated to use centralized hash utilities
- No SHA-256 usage is found in password storage or authentication credential handling code paths
- Test suite includes verification that equivalent data structures produce identical hash outputs via stable serialization
- Code comments document the cryptographic purpose of each hash operation

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All SHA-256 hashing operations must route through centralized utilities, and password hashing MUST use dedicated libraries. Violations block CI pipeline and code review merge.
</enforcement>