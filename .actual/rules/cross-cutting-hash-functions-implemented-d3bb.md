# Adopt Node.js crypto Module for SHA-256 Hashing in Core Libraries: Hash Functions Implemented

These rules are ALWAYS ACTIVE for all core library modules under `src/core/` and public API contracts requiring hash-based identification, data integrity verification workflows, and content-addressable storage implementations.

### Rules

- **R-HASH-001** SHOULD: Hash functions SHOULD be implemented in dedicated utility modules (e.g., src/core/hash.ts) for reusability.
- **R-HASH-002** MUST: All cryptographic hashing in core libraries MUST use Node.js `crypto.createHash('sha256')` for data integrity and content addressing.
- **R-HASH-003** MUST: Structured data hashing implementations MUST use deterministic serialization (stableJson or equivalent) to ensure consistent hash outputs across property ordering variations.
- **R-HASH-004** MUST: Public API contracts MUST expose sha256 functions for external consumers requiring compatible hash generation.
- **R-HASH-005** MUST NOT: Password hashing MUST NOT use SHA-256; use dedicated algorithms like bcrypt or argon2 instead.
- **R-HASH-006** MUST NOT: HMAC or keyed hash operations MUST NOT use createHash; use `crypto.createHmac()` instead.
- **R-HASH-007** MUST NOT: Non-cryptographic hash functions (e.g., xxHash, MurmurHash) MUST NOT be used in core library modules requiring collision resistance.
- **R-HASH-008** MUST NOT: Third-party cryptographic libraries (e.g., crypto-js, noble-hashes) MUST NOT be introduced for core hashing operations.

### Verify

```bash
# Verify SHA-256 usage in core libraries
grep -r "createHash('sha256')" src/core/ --include='*.ts'

# Verify crypto module imports
grep -r "from 'crypto'" src/core/ --include='*.ts'

# Verify deterministic serialization pattern
grep -r "stableJson" src/core/ --include='*.ts'

# Verify no alternative hash libraries in core
grep -r "crypto-js\|noble-hashes\|xxHash\|MurmurHash" src/core/ --include='*.ts'

# Verify no SHA-256 usage for password hashing
grep -r "createHash('sha256')" src/core/ --include='*.ts' | grep -i "password\|pwd\|pass"
```

**Accept when:**
- All core library modules use `crypto.createHash('sha256')` for cryptographic hashing operations
- Public API contracts expose sha256 functions for external consumers
- Structured data hashing implementations use stableJson or equivalent deterministic serialization
- No third-party cryptographic libraries are imported in core modules
- Password hashing uses dedicated algorithms (bcrypt, argon2) not SHA-256
- HMAC operations use `crypto.createHmac()` not `createHash()`

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement.
</enforcement>