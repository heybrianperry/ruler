# Standardize SHA-256 Hashing via Node.js Crypto Module for Data Integrity: Cryptographic Hashing Operations

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files in the codebase that perform cryptographic hashing operations for data integrity verification, content addressing, fingerprinting, or cache key generation.

### Rules

- **R-HASH-001** MUST: All cryptographic hashing operations MUST use the Node.js crypto module's createHash function via centralized utilities in src/core/hash.ts rather than direct crypto.createHash calls.
- **R-HASH-002** MUST: SHA-256 MUST be the standardized hashing algorithm for data integrity, content-addressable storage, cryptographic fingerprinting, and cache key generation use cases.
- **R-HASH-003** MUST: Complex objects MUST be serialized using the stableJson utility before hashing to ensure deterministic outputs for equivalent data structures.
- **R-HASH-004** MUST: SHA-256 MUST NOT be used for password hashing or credential storage; use bcrypt, argon2, or scrypt instead.
- **R-HASH-005** SHOULD: Code comments SHOULD document the purpose of each hash operation to clarify whether cryptographic properties are required for the specific use case.
- **R-HASH-006** SHOULD: Hash results for immutable data structures SHOULD be cached to avoid redundant computation in performance-sensitive paths.
- **R-HASH-007** MAY: Non-cryptographic hash functions (xxHash, MurmurHash) MAY be used for performance-critical internal use cases where adversarial input is not a concern, with documented exception approval.

### Verify

```bash
# Detect direct crypto.createHash calls outside src/core/hash.ts
grep -r "createHash" --include="*.ts" --include="*.js" | grep -v "src/core/hash.ts" | grep -v "test" | grep -v "spec"

# Detect non-SHA-256 usage in crypto.createHash calls
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
Claude Code MUST NOT skip or defer verification of these rules. CI pipeline MUST fail on detection of direct crypto.createHash calls outside core infrastructure modules. Code review MUST block merge requests that use SHA-256 for password storage or authentication. Security team MUST conduct quarterly audits of cryptographic primitive usage.
</enforcement>