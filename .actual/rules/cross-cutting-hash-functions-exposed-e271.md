# Standardize SHA-256 Hashing via Node.js Crypto Module for Data Integrity: Hash Functions Exposed

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files in the codebase that perform cryptographic hashing operations, particularly those handling data integrity verification, content-addressable storage, and deterministic content fingerprinting.

### Rules

- **R-HASH-001** MUST: Hash functions MUST be exposed through public API contracts (sha256, stableJson) for consistent usage across the codebase.
- **R-HASH-002** MUST: All cryptographic hash operations MUST use the Node.js crypto module's createHash('sha256') exclusively, never external hashing libraries.
- **R-HASH-003** MUST: The sha256 and stableJson functions MUST be imported exclusively from src/core/hash.ts by all consumers.
- **R-HASH-004** MUST: Stable JSON serialization MUST be applied to objects before passing to sha256 function to guarantee deterministic outputs.
- **R-HASH-005** SHOULD: Store algorithm metadata (e.g., 'sha256:' prefix) with hash outputs to support future algorithm migration.
- **R-HASH-006** SHOULD: Implement hash result caching for frequently hashed immutable data structures to amortize computational cost.
- **R-HASH-007** MUST: Password hashing operations MUST NOT use SHA-256; use bcrypt, argon2, or similar instead.
- **R-HASH-008** MUST: HMAC operations requiring secret keys MUST NOT use the sha256 function; use appropriate HMAC primitives.
- **R-HASH-009** MUST: Digital signatures requiring asymmetric cryptography MUST NOT use the sha256 function; use appropriate signing primitives.

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
- All cryptographic hash operations use the Node.js crypto module's createHash('sha256') exclusively
- The sha256 and stableJson functions are exported from src/core/hash.ts and imported by all consumers
- Test suite verifies hash determinism across multiple invocations with equivalent data structures
- No external hashing libraries (crypto-js, forge, etc.) are present in production dependencies
- No direct crypto.createHash usage exists outside the src/core/hash.ts module
- All hash operations apply stableJson serialization before hashing
- Integration tests validate hash determinism across service boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Static analysis scanning for direct crypto.createHash usage outside core/hash.ts, dependency audits for external cryptographic libraries, and integration tests for hash determinism are mandatory before code acceptance.
</enforcement>