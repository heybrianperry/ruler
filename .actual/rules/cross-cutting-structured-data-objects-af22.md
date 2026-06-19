# Adopt Node.js crypto Module for SHA-256 Hashing in Core Libraries: Structured Data Objects

These rules are ALWAYS ACTIVE for all core library modules under `src/core/`, public API contracts requiring hash-based identification, data integrity verification workflows, and content-addressable storage implementations.

### Rules

- **R-CRYPTO-001** MUST: Structured data objects MUST be serialized using stableJson before hashing to ensure deterministic output.
- **R-CRYPTO-002** MUST: All core library modules MUST use `crypto.createHash('sha256')` for cryptographic hashing operations requiring data integrity and content addressing.
- **R-CRYPTO-003** MUST: Public API contracts MUST expose sha256 functions for external consumers requiring compatible hash generation.
- **R-CRYPTO-004** SHOULD: Hash utilities SHOULD be centralized in dedicated modules (e.g., `src/core/hash.ts`) to avoid duplication and enable consistent updates.
- **R-CRYPTO-005** SHOULD: Hash-intensive code paths SHOULD implement caching strategies for frequently hashed data to mitigate SHA-256 computational cost.
- **R-CRYPTO-006** MAY: Non-cryptographic hash functions (e.g., xxHash, MurmurHash) MAY be used only for performance-critical scenarios like hash tables where collision resistance is not required.

### Verify

```bash
# Verify crypto module imports in core libraries
grep -r "from 'crypto'" src/core/ --include='*.ts'

# Verify SHA-256 hash creation patterns
grep -r "createHash('sha256')" src/core/ --include='*.ts'

# Verify stableJson serialization usage
grep -r "stableJson" src/core/ --include='*.ts'

# Verify no alternative cryptographic hash libraries
grep -r "crypto-js\|noble-hashes" src/core/ --include='*.ts' && echo "FAIL: Alternative crypto libraries detected" || echo "PASS: No alternative crypto libraries"

# Verify no non-sha256 hash algorithms in core
grep -r "createHash('sha512')\|createHash('sha3')\|createHash('md5')" src/core/ --include='*.ts' && echo "FAIL: Non-SHA256 algorithms detected" || echo "PASS: Only SHA256 used"
```

**Accept when:**
- All core library modules use `crypto.createHash('sha256')` for cryptographic hashing operations
- Public API contracts expose sha256 functions for external consumers
- Structured data hashing implementations use stableJson or equivalent deterministic serialization
- No alternative cryptographic hash libraries (crypto-js, noble-hashes) are imported in core modules
- No non-SHA256 hash algorithms are used in core library hashing operations
- Hash utilities are centralized in dedicated modules with clear documentation

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement. Violations MUST be caught during static analysis and code review before merge.
</enforcement>