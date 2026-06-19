# Standardize SHA-256 Hashing via Node.js Crypto Module for Data Integrity: Hash Generation Logic

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files in the codebase that perform cryptographic hashing operations for data integrity verification, content-addressable storage, or deterministic content fingerprinting.

### Rules

- **R-HASH-001** MUST: Hash generation logic MUST be centralized in core infrastructure modules (e.g., `src/core/hash.ts`) to prevent implementation drift and ensure consistent security posture.
- **R-HASH-002** MUST: All cryptographic hash operations MUST use the Node.js `crypto` module's `createHash('sha256')` exclusively; no external hashing libraries (crypto-js, forge, etc.) are permitted in production code.
- **R-HASH-003** MUST: The `sha256` and `stableJson` functions MUST be exported from `src/core/hash.ts` and imported by all consumers; direct `crypto.createHash` usage outside this module is prohibited.
- **R-HASH-004** MUST: Stable JSON serialization MUST be applied to all objects before hashing to guarantee deterministic outputs across different execution contexts and property ordering variations.
- **R-HASH-005** SHOULD: Hash results SHOULD be stored with algorithm metadata (e.g., `'sha256:'` prefix) to support future algorithm migration without breaking existing hashes.
- **R-HASH-006** SHOULD: Hash result caching SHOULD be implemented for frequently hashed immutable data structures to amortize computational cost in performance-critical paths.
- **R-HASH-007** MAY: Non-cryptographic hash functions (xxHash, MurmurHash) MAY be used only for performance-critical internal caching where collision attacks are not a threat model concern and security-sensitive integrity verification is not required.

### Verify

```bash
# Verify all createHash usage is centralized in src/core/hash.ts
grep -r "createHash" --include="*.ts" --include="*.js" | grep -v "src/core/hash.ts" | grep -v "test" || echo "All createHash usage centralized"

# Count imports of sha256 from core/hash module
grep -r "import.*sha256.*from.*['\"].*core/hash" --include="*.ts" --include="*.js" | wc -l

# Verify SHA-256 produces correct 64-character hex output
node -e "const crypto = require('crypto'); const hash = crypto.createHash('sha256'); hash.update('test'); console.log(hash.digest('hex'));" | grep -q "^[a-f0-9]{64}$" && echo "SHA-256 verification passed"

# Check for external cryptographic libraries in dependencies
grep -E "crypto-js|forge|tweetnacl" package.json || echo "No external crypto libraries detected"
```

**Accept when:**
- All cryptographic hash operations use the Node.js `crypto` module's `createHash('sha256')` exclusively
- The `sha256` and `stableJson` functions are exported from `src/core/hash.ts` and imported by all consumers
- No direct `crypto.createHash` usage exists outside the `src/core/hash.ts` module (excluding test files)
- Test suite verifies hash determinism across multiple invocations with equivalent data structures
- No external hashing libraries (crypto-js, forge, etc.) are present in production dependencies
- Hash operations for data integrity verification, content-addressable storage, and deterministic fingerprinting all route through centralized core infrastructure

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis scanning for direct `crypto.createHash` usage outside `src/core/hash.ts`, dependency audits for external cryptographic libraries, and integration tests validating hash determinism are mandatory before code acceptance. Violations require refactoring to use centralized hash utilities before merge approval.
</enforcement>