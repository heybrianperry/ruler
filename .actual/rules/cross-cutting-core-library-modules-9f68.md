# Adopt Node.js crypto Module for SHA-256 Hashing in Core Libraries: Core Library Modules

These rules are ALWAYS ACTIVE for all core library modules under `src/core/` that implement cryptographic hash operations, public API contracts requiring hash-based identification, data integrity verification workflows, and content-addressable storage implementations.

### Rules

- **R-CRYPTO-001** MUST: Core library modules MUST use the Node.js `crypto` module for cryptographic hash operations.
- **R-CRYPTO-002** MUST: All SHA-256 hash generation in core libraries MUST use `crypto.createHash('sha256')`.
- **R-CRYPTO-003** MUST: Structured data hashing implementations MUST use deterministic serialization (e.g., `stableJson`) to ensure consistent hash outputs across identical inputs.
- **R-CRYPTO-004** MUST: Public API contracts MUST expose `sha256` functions for external consumers requiring compatible hash generation.
- **R-CRYPTO-005** SHOULD: Hash utilities SHOULD be centralized in dedicated modules (e.g., `src/core/hash.ts`) to avoid duplication and enable consistent updates.
- **R-CRYPTO-006** MAY: Non-cryptographic hash functions (e.g., xxHash, MurmurHash) MAY be used only for performance-critical scenarios like hash tables where collision resistance is not required, with explicit documentation of the trade-off.

### Verify

```bash
# Verify crypto module imports in core libraries
grep -r "from 'crypto'" src/core/ --include='*.ts'

# Verify SHA-256 usage pattern
grep -r "createHash('sha256')" src/core/ --include='*.ts'

# Verify deterministic serialization for structured data
grep -r "stableJson" src/core/ --include='*.ts'

# Verify no alternative cryptographic libraries in core
grep -r "crypto-js\|noble-hashes" src/core/ --include='*.ts' && echo "FAIL: Alternative crypto libraries detected" || echo "PASS: No alternative crypto libraries"

# Verify no non-standard hash algorithms in core
grep -r "createHash('" src/core/ --include='*.ts' | grep -v "createHash('sha256')" && echo "FAIL: Non-standard hash algorithms detected" || echo "PASS: Only SHA-256 used"
```

**Accept when:**
- All core library modules use `crypto.createHash('sha256')` for cryptographic hashing operations
- Public API contracts expose `sha256` functions for external consumers
- Structured data hashing implementations use `stableJson` or equivalent deterministic serialization
- No alternative cryptographic libraries (crypto-js, noble-hashes) are imported in core libraries
- All hash algorithm calls use only SHA-256 in core library scope

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis scanning for crypto module imports and createHash usage patterns is mandatory. Code review must verify hash algorithm consistency. Unit tests must validate deterministic hash outputs for identical inputs. CI pipeline must fail if non-standard hash implementations are detected in core libraries.
</enforcement>