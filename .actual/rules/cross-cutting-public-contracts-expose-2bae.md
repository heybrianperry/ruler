# Adopt Node.js crypto Module for SHA-256 Hashing in Core Libraries: Public Contracts Expose

These rules are ALWAYS ACTIVE for all core library modules under `src/core/` that expose public API contracts requiring hash-based identification, data integrity verification, and content-addressable storage implementations.

### Rules

- **R-CRYPTO-001** MUST: Public API contracts MUST expose sha256 hashing functions for external consumers.
- **R-CRYPTO-002** MUST: All core library modules use `crypto.createHash('sha256')` for cryptographic hashing operations.
- **R-CRYPTO-003** MUST: Structured data hashing implementations use stableJson or equivalent deterministic serialization to ensure consistent hash outputs.
- **R-CRYPTO-004** MUST: Hash utilities be centralized in dedicated modules (e.g., `src/core/hash.ts`) to avoid duplication and enable consistent updates.
- **R-CRYPTO-005** SHOULD: Document the hash algorithm choice and serialization strategy in API documentation for consumers implementing compatible systems.

### Verify

```bash
# Verify crypto module imports in core libraries
grep -r "from 'crypto'" src/core/ --include='*.ts'

# Verify SHA-256 hash creation patterns
grep -r "createHash('sha256')" src/core/ --include='*.ts'

# Verify stableJson serialization usage
grep -r "stableJson" src/core/ --include='*.ts'

# Verify public API exports of sha256 functions
grep -r "export.*sha256" src/core/ --include='*.ts'
```

**Accept when:**
- All core library modules use `crypto.createHash('sha256')` for cryptographic hashing operations
- Public API contracts expose sha256 functions for external consumers
- Structured data hashing implementations use stableJson or equivalent deterministic serialization
- Hash utilities are centralized in dedicated modules to avoid duplication
- No alternative cryptographic hash libraries are introduced in core libraries

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis scanning for crypto module imports and createHash usage patterns is mandatory. Code review must verify hash algorithm consistency. Unit tests must validate deterministic hash outputs for identical inputs. CI pipeline must fail if non-standard hash implementations are detected in core libraries.
</enforcement>