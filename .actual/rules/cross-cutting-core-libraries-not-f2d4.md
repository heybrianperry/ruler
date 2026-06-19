# Adopt Node.js crypto Module for SHA-256 Hashing in Core Libraries: Core Libraries Not

These rules are ALWAYS ACTIVE for all core library modules under `src/core/` and public API contracts requiring hash-based identification, data integrity verification workflows, and content-addressable storage implementations.

### Rules

- **R-CRYPTO-001** MUST_NOT: Core libraries MUST NOT implement custom cryptographic hash algorithms.
- **R-CRYPTO-002** MUST: All core library modules use `crypto.createHash('sha256')` for cryptographic hashing operations.
- **R-CRYPTO-003** MUST: Public API contracts expose `sha256` functions for external consumers.
- **R-CRYPTO-004** MUST: Structured data hashing implementations use `stableJson` or equivalent deterministic serialization.
- **R-CRYPTO-005** MUST_NOT: Core libraries MUST NOT use third-party cryptographic libraries (e.g., crypto-js, noble-hashes) for SHA-256 hashing.
- **R-CRYPTO-006** MUST_NOT: Core libraries MUST NOT use non-cryptographic hash functions (e.g., xxHash, MurmurHash) for data integrity or security-sensitive use cases.
- **R-CRYPTO-007** SHOULD: Centralize hash utilities in `src/core/hash.ts` or similar modules to avoid duplication and enable consistent updates.
- **R-CRYPTO-008** SHOULD: Document the hash algorithm choice and serialization strategy in API documentation for consumers implementing compatible systems.

### Verify

```bash
# Verify crypto.createHash('sha256') usage in core libraries
grep -r "createHash('sha256')" src/core/ --include='*.ts'

# Verify crypto module imports in core libraries
grep -r "from 'crypto'" src/core/ --include='*.ts'

# Verify stableJson serialization usage
grep -r "stableJson" src/core/ --include='*.ts'

# Verify no custom hash implementations
grep -r "function.*hash\|const.*hash.*=" src/core/ --include='*.ts' | grep -v "createHash\|stableJson"

# Verify no third-party crypto library imports
grep -r "from 'crypto-js'\|from 'noble-hashes'\|import.*crypto-js\|import.*noble-hashes" src/core/ --include='*.ts'
```

**Accept when:**
- All core library modules use `crypto.createHash('sha256')` for cryptographic hashing operations
- Public API contracts expose `sha256` functions for external consumers
- Structured data hashing implementations use `stableJson` or equivalent deterministic serialization
- No custom cryptographic hash algorithms are implemented in core libraries
- No third-party cryptographic libraries are used for SHA-256 hashing
- Hash utilities are centralized in dedicated modules
- Unit tests validate deterministic hash outputs for identical inputs

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis scanning for crypto module imports and createHash usage patterns is mandatory. Code review must verify hash algorithm consistency. CI pipeline must fail if non-standard hash implementations are detected in core libraries.
</enforcement>