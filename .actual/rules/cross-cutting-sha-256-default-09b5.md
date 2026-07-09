# Standardize SHA-256 Hashing via Node.js Crypto Module for Data Integrity: Sha 256 Default

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files in the codebase that perform cryptographic hashing, data integrity verification, content addressing, or fingerprinting operations.

### Rules

- **R-SHA256-001** MUST: SHA-256 MUST be the default hashing algorithm for data integrity and content addressing use cases.
- **R-SHA256-002** MUST: All direct `crypto.createHash()` calls outside `src/core/hash.ts` MUST be migrated to use centralized hash utilities from `src/core/hash.ts`.
- **R-SHA256-003** MUST: SHA-256 MUST NOT be used for password hashing or credential storage; use bcrypt, argon2, or scrypt instead.
- **R-SHA256-004** SHOULD: When hashing complex objects, use the `stableJson` utility to serialize data structures in canonical form before hashing to ensure deterministic outputs.
- **R-SHA256-005** SHOULD: Document the purpose of each hash operation in code comments to clarify whether cryptographic properties are required for the specific use case.
- **R-SHA256-006** MAY: Non-cryptographic hash functions (xxHash, MurmurHash) are acceptable only for performance-critical internal use cases where adversarial input is not a concern and security properties are not required.

### Verify

```bash
# Detect direct crypto.createHash calls outside approved core modules
grep -r "createHash" --include="*.ts" --include="*.js" | grep -v "src/core/hash.ts" | grep -v "test" | grep -v "spec"

# Detect SHA-256 usage with non-standard algorithms
grep -r "crypto\.createHash" --include="*.ts" --include="*.js" | grep -v "'sha256'" | grep -v '"sha256"'

# Verify deterministic hashing behavior tests exist
npm test -- --grep "hash.*deterministic" || echo 'Add tests for deterministic hashing behavior'
```

**Accept when:**
- All direct `crypto.createHash()` calls outside `src/core/hash.ts` are migrated to use centralized hash utilities
- No SHA-256 usage is found in password storage or authentication credential handling code paths
- Test suite includes verification that equivalent data structures produce identical hash outputs via stable serialization
- Code comments document the cryptographic purpose of each hash operation
- No violations of R-SHA256-003 are detected in security-focused code review

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All hashing operations must be audited against R-SHA256-001 through R-SHA256-006. Violations of MUST-level rules (R-SHA256-001, R-SHA256-002, R-SHA256-003) block acceptance.
</enforcement>