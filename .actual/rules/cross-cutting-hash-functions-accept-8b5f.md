# Standardize SHA-256 Hashing via Node.js Crypto Module for Data Integrity: Hash Functions Accept

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files in the codebase that perform cryptographic hashing operations for data integrity, content addressing, or fingerprinting.

### Rules

- **R-HASH-001** MUST: Hash functions MUST accept input data in a normalized form to ensure deterministic output.
- **R-HASH-002** MUST: All direct `crypto.createHash` calls outside `src/core/hash.ts` MUST be migrated to use centralized hash utilities from the core module.
- **R-HASH-003** MUST: SHA-256 MUST NOT be used for password storage or authentication credential handling; use bcrypt, argon2, or scrypt instead.
- **R-HASH-004** SHOULD: When hashing complex objects, use the `stableJson` utility to serialize data structures in canonical form before hashing to ensure deterministic outputs.
- **R-HASH-005** SHOULD: Document the purpose of each hash operation in code comments to clarify whether cryptographic properties are required for the specific use case.
- **R-HASH-006** MAY: Cache hash results for immutable data structures to avoid redundant computation in performance-sensitive paths.

### Verify

```bash
# Detect direct crypto.createHash calls outside approved core modules
grep -r "createHash" --include="*.ts" --include="*.js" | grep -v "src/core/hash.ts" | grep -v "test" | grep -v "spec"

# Detect non-SHA-256 hashing algorithms
grep -r "crypto\.createHash" --include="*.ts" --include="*.js" | grep -v "'sha256'" | grep -v '"sha256"'

# Verify deterministic hashing tests exist
npm test -- --grep "hash.*deterministic" || echo 'Add tests for deterministic hashing behavior'
```

**Accept when:**
- All direct `crypto.createHash` calls outside `src/core/hash.ts` are migrated to use centralized hash utilities
- No SHA-256 usage is found in password storage or authentication credential handling code paths
- Test suite includes verification that equivalent data structures produce identical hash outputs via stable serialization
- Code review checklist confirms hashing use cases are appropriate for SHA-256
- Security-focused test suite validates deterministic hashing behavior and prevents password hashing misuse

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement. Violations MUST be flagged and remediated before merge.
</enforcement>