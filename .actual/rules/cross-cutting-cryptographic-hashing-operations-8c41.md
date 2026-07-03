# Standardize SHA-256 Hashing via Node.js Crypto Module for Data Integrity: Cryptographic Hashing Operations

These rules are ALWAYS ACTIVE for all cryptographic hashing operations, data integrity verification, content-addressable storage key generation, deterministic content fingerprinting, and core infrastructure hashing utilities.

### Rules

- **R-CRYPTO-001** MUST: All cryptographic hashing operations MUST use the Node.js crypto module's createHash function with SHA-256 algorithm exclusively.
- **R-CRYPTO-002** MUST: All hash operations MUST import and use the sha256 and stableJson functions exclusively from src/core/hash.ts.
- **R-CRYPTO-003** MUST: All objects MUST be serialized using stableJson before passing to the sha256 function to guarantee deterministic outputs.
- **R-CRYPTO-004** MUST: Hash outputs MUST include algorithm metadata (e.g., 'sha256:' prefix) to support future algorithm migration.
- **R-CRYPTO-005** SHOULD: Frequently hashed immutable data structures SHOULD implement hash result caching to amortize computational cost.
- **R-CRYPTO-006** MUST NOT: Password hashing operations MUST NOT use SHA-256; use bcrypt, argon2, or similar instead.
- **R-CRYPTO-007** MUST NOT: HMAC operations requiring secret keys MUST NOT use the centralized sha256 utility.
- **R-CRYPTO-008** MUST NOT: Digital signatures requiring asymmetric cryptography MUST NOT use the centralized sha256 utility.
- **R-CRYPTO-009** MUST NOT: Hash functions for hash tables or non-cryptographic purposes MUST NOT use the centralized sha256 utility.

### Verify

```bash
# Verify all createHash usage is centralized in src/core/hash.ts
grep -r "createHash" --include="*.ts" --include="*.js" | grep -v "src/core/hash.ts" | grep -v "test" || echo "All createHash usage centralized"

# Count imports of sha256 from core/hash module
grep -r "import.*sha256.*from.*['\"].*core/hash" --include="*.ts" --include="*.js" | wc -l

# Verify SHA-256 produces correct 64-character hex output
node -e "const crypto = require('crypto'); const hash = crypto.createHash('sha256'); hash.update('test'); console.log(hash.digest('hex'));" | grep -q "^[a-f0-9]{64}$" && echo "SHA-256 verification passed"

# Verify no external cryptographic libraries in dependencies
grep -E "crypto-js|forge|tweetnacl" package.json || echo "No external crypto libraries detected"
```

**Accept when:**
- All cryptographic hash operations use the Node.js crypto module's createHash('sha256') exclusively
- The sha256 and stableJson functions are exported from src/core/hash.ts and imported by all consumers
- Test suite verifies hash determinism across multiple invocations with equivalent data structures
- No external hashing libraries (crypto-js, forge, etc.) are present in production dependencies
- No direct crypto.createHash usage exists outside the src/core/hash.ts module
- All hash operations include algorithm metadata prefixes for migration support

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Static analysis scanning for direct crypto.createHash usage outside core/hash.ts, dependency audits for external cryptographic libraries, code review validation of centralized hash utility imports, and integration tests for hash determinism are mandatory before accepting any changes to cryptographic hashing operations.
</enforcement>