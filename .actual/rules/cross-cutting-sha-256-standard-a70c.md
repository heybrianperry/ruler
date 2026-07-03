# Standardize SHA-256 Hashing via Node.js Crypto Module for Data Integrity: Sha 256 Standard

These rules are ALWAYS ACTIVE for all cryptographic hash operations, data integrity verification, content-addressable storage key generation, and deterministic content fingerprinting across the codebase.

### Rules

- **R-SHA256-001** MUST: SHA-256 MUST be the standard hashing algorithm for data integrity and content addressing.
- **R-SHA256-002** MUST: All cryptographic hash operations MUST use the Node.js crypto module's `createHash('sha256')` exclusively, imported only from `src/core/hash.ts`.
- **R-SHA256-003** MUST: Stable JSON serialization MUST be applied to all objects before hashing to ensure deterministic outputs across different execution contexts.
- **R-SHA256-004** MUST: Hash operations MUST NOT bypass the centralized `sha256` and `stableJson` functions exported from `src/core/hash.ts`.
- **R-SHA256-005** SHOULD: Store algorithm metadata (e.g., 'sha256:' prefix) with hash outputs to support future algorithm migration.
- **R-SHA256-006** SHOULD: Implement hash result caching for frequently hashed immutable data structures to amortize computational cost.
- **R-SHA256-007** MAY: Non-cryptographic hash functions (xxHash, MurmurHash) MAY be used only for performance-critical internal caching where collision attacks are not a threat model concern.

### Verify

```bash
# Verify all createHash usage is centralized in src/core/hash.ts
grep -r "createHash" --include="*.ts" --include="*.js" | grep -v "src/core/hash.ts" | grep -v "test" || echo "All createHash usage centralized"

# Count sha256 imports from core/hash module
grep -r "import.*sha256.*from.*['\"].*core/hash" --include="*.ts" --include="*.js" | wc -l

# Verify SHA-256 produces correct 64-character hex output
node -e "const crypto = require('crypto'); const hash = crypto.createHash('sha256'); hash.update('test'); console.log(hash.digest('hex'));" | grep -q "^[a-f0-9]{64}$" && echo "SHA-256 verification passed"

# Check for external cryptographic libraries in dependencies
grep -E "crypto-js|forge|tweetnacl" package.json || echo "No external crypto libraries detected"
```

**Accept when:**
- All cryptographic hash operations use the Node.js crypto module's `createHash('sha256')` exclusively
- The `sha256` and `stableJson` functions are exported from `src/core/hash.ts` and imported by all consumers
- No direct `crypto.createHash` usage exists outside `src/core/hash.ts` (excluding test files)
- Test suite verifies hash determinism across multiple invocations with equivalent data structures
- No external hashing libraries (crypto-js, forge, etc.) are present in production dependencies
- Hash outputs include algorithm metadata prefix for future migration support
- Code review checklist confirms hash operations use centralized utilities

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Static analysis scanning for direct crypto.createHash usage outside core/hash.ts, dependency audits for external cryptographic libraries, and integration tests validating hash determinism are mandatory before code acceptance.
</enforcement>