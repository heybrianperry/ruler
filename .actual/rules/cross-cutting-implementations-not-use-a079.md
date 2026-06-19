# Standardize SHA-256 Hashing via Node.js Crypto Module for Data Integrity: Implementations Not Use

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files in the codebase that perform cryptographic hashing operations for data integrity verification, content-addressable storage, and deterministic content fingerprinting.

### Rules

- **R-SHA256-001** MUST_NOT: Implementations MUST NOT use external hashing libraries when the Node.js crypto module provides equivalent functionality.
- **R-SHA256-002** MUST: All cryptographic hash operations MUST use the Node.js crypto module's `createHash('sha256')` exclusively.
- **R-SHA256-003** MUST: The `sha256` and `stableJson` functions MUST be exported from `src/core/hash.ts` and imported by all consumers.
- **R-SHA256-004** MUST: Always apply `stableJson` serialization to objects before passing to `sha256` function to guarantee deterministic outputs.
- **R-SHA256-005** SHOULD: Store algorithm metadata (e.g., 'sha256:' prefix) with hash outputs to support future algorithm migration.
- **R-SHA256-006** SHOULD: Consider implementing hash result caching for frequently hashed immutable data structures to amortize computational cost.

### Verify

```bash
# Verify all createHash usage is centralized in src/core/hash.ts
grep -r "createHash" --include="*.ts" --include="*.js" | grep -v "src/core/hash.ts" | grep -v "test" || echo "All createHash usage centralized"

# Count imports of sha256 from core/hash module
grep -r "import.*sha256.*from.*['\"].*core/hash" --include="*.ts" --include="*.js" | wc -l

# Verify SHA-256 produces correct 64-character hex output
node -e "const crypto = require('crypto'); const hash = crypto.createHash('sha256'); hash.update('test'); console.log(hash.digest('hex'));" | grep -q "^[a-f0-9]{64}$" && echo "SHA-256 verification passed"

# Check for external hashing libraries in dependencies
grep -E "crypto-js|forge|tweetnacl" package.json || echo "No external hashing libraries detected"
```

**Accept when:**
- All cryptographic hash operations use the Node.js crypto module's `createHash('sha256')` exclusively
- The `sha256` and `stableJson` functions are exported from `src/core/hash.ts` and imported by all consumers
- Test suite verifies hash determinism across multiple invocations with equivalent data structures
- No external hashing libraries (crypto-js, forge, etc.) are present in production dependencies
- No direct `crypto.createHash` usage exists outside the `src/core/hash.ts` module (excluding tests)

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Static analysis scanning for direct crypto.createHash usage outside core/hash.ts, dependency audits for external cryptographic libraries, and integration tests validating hash determinism are mandatory before code acceptance.
</enforcement>