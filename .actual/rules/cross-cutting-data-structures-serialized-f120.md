# Standardize SHA-256 Hashing via Node.js Crypto Module for Data Integrity: Data Structures Serialized

These rules are ALWAYS ACTIVE for all data integrity verification operations, content-addressable storage key generation, deterministic content fingerprinting, cryptographic hash generation for non-password use cases, and core infrastructure hashing utilities.

### Rules

- **R-SHA256-001** MUST: Data structures MUST be serialized using stable JSON serialization (stableJson) before hashing to ensure deterministic outputs.
- **R-SHA256-002** MUST: All cryptographic hash operations MUST use the Node.js crypto module's createHash('sha256') exclusively.
- **R-SHA256-003** MUST: The sha256 and stableJson functions MUST be exported from src/core/hash.ts and imported by all consumers.
- **R-SHA256-004** MUST: Hash operations MUST NOT bypass core infrastructure utilities.
- **R-SHA256-005** SHOULD: Store algorithm metadata (e.g., 'sha256:' prefix) with hash outputs to support future algorithm migration.
- **R-SHA256-006** SHOULD: Consider implementing hash result caching for frequently hashed immutable data structures to amortize computational cost.

### Verify

```bash
# Verify all createHash usage is centralized in core/hash.ts
grep -r "createHash" --include="*.ts" --include="*.js" | grep -v "src/core/hash.ts" | grep -v "test" || echo "All createHash usage centralized"

# Verify sha256 imports from core/hash
grep -r "import.*sha256.*from.*['\"].*core/hash" --include="*.ts" --include="*.js" | wc -l

# Verify SHA-256 produces correct output format
node -e "const crypto = require('crypto'); const hash = crypto.createHash('sha256'); hash.update('test'); console.log(hash.digest('hex'));" | grep -q "^[a-f0-9]{64}$" && echo "SHA-256 verification passed"
```

**Accept when:**
- All cryptographic hash operations use the Node.js crypto module's createHash('sha256') exclusively
- The sha256 and stableJson functions are exported from src/core/hash.ts and imported by all consumers
- Test suite verifies hash determinism across multiple invocations with equivalent data structures
- No external hashing libraries (crypto-js, forge, etc.) are present in production dependencies
- No direct crypto.createHash usage exists outside approved modules

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Static analysis scanning for direct crypto.createHash usage outside core/hash.ts, dependency audits for external cryptographic libraries, code review checklists, and integration tests validating hash determinism are mandatory before acceptance.
</enforcement>