# Standardize SHA-256 Hashing via Node.js Crypto Module for Data Integrity: Hash Utility Functions

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files that require cryptographic hashing for data integrity, content addressing, or fingerprinting operations.

### Rules

- **R-HASH-001** MUST: Hash utility functions MUST be centralized in core infrastructure modules (src/core/hash.ts) to ensure consistent implementation across the codebase.
- **R-HASH-002** MUST: All SHA-256 hashing operations MUST use the centralized hash utilities from src/core/hash.ts rather than directly calling crypto.createHash().
- **R-HASH-003** MUST: SHA-256 MUST NOT be used for password hashing or credential storage; use bcrypt, argon2, or scrypt instead.
- **R-HASH-004** SHOULD: Complex objects SHOULD be serialized using the stableJson utility before hashing to ensure deterministic outputs for equivalent data structures.
- **R-HASH-005** SHOULD: Hash operations SHOULD be documented with code comments clarifying whether cryptographic properties are required for the specific use case.
- **R-HASH-006** MAY: Non-cryptographic hash functions (xxHash, MurmurHash) MAY be used for performance-critical internal use cases where adversarial input is not a concern and security properties are not required, subject to exception approval.

### Verify

```bash
# Detect direct crypto.createHash calls outside core infrastructure
grep -r "createHash" --include="*.ts" --include="*.js" | grep -v "src/core/hash.ts" | grep -v "test" | grep -v "spec"

# Detect non-SHA-256 crypto.createHash usage
grep -r "crypto\.createHash" --include="*.ts" --include="*.js" | grep -v "'sha256'" | grep -v '"sha256"'

# Verify deterministic hashing tests exist
npm test -- --grep "hash.*deterministic" || echo 'Add tests for deterministic hashing behavior'
```

**Accept when:**
- All direct crypto.createHash calls outside src/core/hash.ts are migrated to use centralized hash utilities
- No SHA-256 usage is found in password storage or authentication credential handling code paths
- Test suite includes verification that equivalent data structures produce identical hash outputs via stable serialization
- Code review checklist confirms hashing use cases are appropriate for SHA-256
- Security-focused test suite validates deterministic hashing behavior and prevents password hashing misuse

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement.
</enforcement>