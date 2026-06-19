# Adopt Node.js crypto Module for SHA-256 Hashing in Core Libraries: Sha 256 Used

These rules are ALWAYS ACTIVE for all core library modules under `src/core/` and public API contracts requiring hash-based identification, data integrity verification workflows, and content-addressable storage implementations.

### Rules

- **R-SHA256-001** MUST: SHA-256 MUST be used as the standard hash algorithm via `crypto.createHash('sha256')` in all core library modules.
- **R-SHA256-002** MUST: Structured data hashing implementations MUST use deterministic serialization (stableJson or equivalent) to ensure consistent hash outputs regardless of property ordering.
- **R-SHA256-003** MUST: Hash utilities MUST be centralized in dedicated modules (e.g., `src/core/hash.ts`) to avoid duplication and enable consistent updates.
- **R-SHA256-004** MUST: Public API contracts MUST expose sha256 functions for external consumers requiring compatible hash generation.
- **R-SHA256-005** SHOULD: Hash-intensive code paths SHOULD be profiled and implement caching strategies for frequently hashed data.
- **R-SHA256-006** SHOULD: Runtime requirements and environment compatibility SHOULD be clearly documented for consumers implementing compatible systems.

### Verify

```bash
# Verify SHA-256 usage in core libraries
grep -r "createHash('sha256')" src/core/ --include='*.ts'

# Verify crypto module imports
grep -r "from 'crypto'" src/core/ --include='*.ts'

# Verify stableJson serialization usage
grep -r "stableJson" src/core/ --include='*.ts'
```

**Accept when:**
- All core library modules use `crypto.createHash('sha256')` for cryptographic hashing operations
- Public API contracts expose sha256 functions for external consumers
- Structured data hashing implementations use stableJson or equivalent deterministic serialization
- Hash utilities are centralized in dedicated modules
- No alternative cryptographic hash implementations are detected in core libraries

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis scanning for crypto module imports and createHash usage patterns is mandatory. Code review must verify hash algorithm consistency. Unit tests must validate deterministic hash outputs for identical inputs. CI pipeline MUST fail if non-standard hash implementations are detected in core libraries.
</enforcement>