# Use Node.js crypto Module with SHA-256 for Cryptographic Hashing: Hash Operations Requiring Cryptographic Strength Use

These rules are ALWAYS ACTIVE for all files implementing cryptographic hash operations, core utility modules providing hash operations, security-sensitive data processing, content addressing, and data integrity verification use cases.

### Rules

- **R-CRYPTO-001** MUST: Hash operations requiring cryptographic strength MUST use SHA-256 as the hash algorithm.
- **R-CRYPTO-002** MUST: All cryptographic hash operations MUST use the Node.js crypto module's `createHash` API with explicit SHA-256 algorithm specification.
- **R-CRYPTO-003** MUST: No usage of deprecated or weak hash algorithms (MD5, SHA-1) is permitted in security-sensitive code paths.
- **R-CRYPTO-004** SHOULD: Hash operations SHOULD be centralized in core utility modules rather than implemented inline throughout the codebase.
- **R-CRYPTO-005** SHOULD: Core hash utility modules SHOULD have comprehensive test coverage demonstrating correct SHA-256 implementation.
- **R-CRYPTO-006** MAY: Non-cryptographic hash operations where performance is critical and security is not required MAY use faster non-crypto hash functions.
- **R-CRYPTO-007** MAY: Password hashing operations MAY use dedicated password hashing algorithms with salt and iteration (bcrypt, argon2) instead of SHA-256.

### Verify

```bash
# Discover the project's static analysis or linting configuration and execute security-focused rules
# to verify crypto module usage patterns
grep -r "createHash" --include="*.js" --include="*.ts" | grep -v node_modules

# Locate the project's test suite and execute tests covering the core hash utility module
# to verify SHA-256 implementation correctness
npm test -- --testPathPattern="hash|crypto"

# Search the codebase for all hash-related operations and verify they route through
# centralized utilities rather than inline crypto module usage
grep -r "require.*crypto\|import.*crypto" --include="*.js" --include="*.ts" | grep -v node_modules | grep -v "core.*util"

# Verify no weak hash algorithms are in use
grep -r "md5\|sha1\|MD5\|SHA1" --include="*.js" --include="*.ts" | grep -v node_modules | grep -v test | grep -v comment
```

**Accept when:**
- All cryptographic hash operations use the crypto module's `createHash` API with SHA-256 algorithm specification
- No usage of deprecated or weak hash algorithms (MD5, SHA-1) exists in security-sensitive code paths
- Hash operations are centralized in core utility modules with test coverage demonstrating correct implementation
- Direct crypto module usage outside approved utility modules is not present or has documented exception approval
- The Node.js version in use supports the required crypto APIs as verified against the project's runtime version specification

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules R-CRYPTO-001 through R-CRYPTO-007 MUST be checked before approving code changes affecting cryptographic hash operations.
</enforcement>