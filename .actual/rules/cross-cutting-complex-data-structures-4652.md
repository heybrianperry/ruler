# Standardize SHA-256 Hashing via Node.js Crypto Module for Data Integrity: Complex Data Structures

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files that perform cryptographic hashing operations, data integrity verification, content addressing, or fingerprinting of data structures.

### Rules

- **R-SHA256-001** SHOULD: Complex data structures SHOULD be serialized using stable JSON serialization before hashing to ensure deterministic outputs for equivalent data structures.
- **R-SHA256-002** MUST: Import hash utilities from `src/core/hash.ts` rather than directly calling `crypto.createHash` to ensure consistent implementation and enable future algorithm updates.
- **R-SHA256-003** MUST: Never use SHA-256 directly for password storage, credential hashing, or authentication. Use dedicated password hashing libraries (bcrypt, argon2, scrypt) with appropriate work factors.
- **R-SHA256-004** SHOULD: Document the purpose of each hash operation in code comments to clarify whether cryptographic properties are required for the specific use case.
- **R-SHA256-005** MAY: Cache hash results for immutable data structures to avoid redundant computation in performance-sensitive paths.
- **R-SHA256-006** MUST: Approved exceptions for non-cryptographic hashing (EXC-001, EXC-002) require explicit documentation and security team review before implementation.

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
- All direct `crypto.createHash` calls outside `src/core/hash.ts` are migrated to use centralized hash utilities from the core module.
- No SHA-256 usage is found in password storage, authentication credential handling, or credential verification code paths.
- Test suite includes verification that equivalent data structures produce identical hash outputs via stable serialization.
- Code review checklist confirms hashing use cases are appropriate for SHA-256 and not misused for password storage.
- All exceptions (EXC-001, EXC-002) are documented with explicit scope, duration, and compensating controls.

<enforcement>
Claude Code MUST NOT skip or defer verification. All direct crypto.createHash calls must be audited and migrated to centralized utilities. Password hashing misuse must be blocked at code review and CI pipeline stages.
</enforcement>