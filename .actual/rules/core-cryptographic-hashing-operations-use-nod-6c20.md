# Use Node.js crypto Module with SHA-256 for Cryptographic Hashing: Cryptographic Hashing Operations Use Node Crypto

These rules are ALWAYS ACTIVE for all cryptographic hashing operations in core utility modules and security-sensitive data processing code paths.

### Rules

- **R-CRYPTO-001** MUST: Cryptographic hashing operations MUST use the Node.js crypto module's createHash API.
- **R-CRYPTO-002** MUST: All createHash calls MUST explicitly specify the SHA-256 algorithm rather than relying on defaults.
- **R-CRYPTO-003** MUST: Hash operations MUST be centralized in core utility modules rather than implemented inline throughout the codebase.
- **R-CRYPTO-004** MUST: No usage of weak hash algorithms (MD5, SHA-1) is permitted in security-sensitive code paths.
- **R-CRYPTO-005** SHOULD: For data integrity use cases, consider whether HMAC (keyed hashing) provides better security properties than plain hashing for the specific threat model.
- **R-CRYPTO-006** MUST: Password hashing operations MUST use dedicated password hashing algorithms (bcrypt, argon2) with salt and iteration, not SHA-256 alone.

### Verify

```bash
# Discover the project's Node.js version specification and verify crypto module API support
grep -E '(node|engines)' package.json | head -5

# Locate and execute the project's test suite for core hash utility module
npm test -- --grep "hash|crypto" 2>/dev/null || echo "(run project test suite manually)"

# Search codebase for all hash-related operations
grep -r "createHash\|crypto\.createHash" --include="*.js" --include="*.ts" . 2>/dev/null | grep -v node_modules | head -20

# Verify no weak hash algorithms in security-sensitive paths
grep -r "md5\|sha1\|MD5\|SHA1" --include="*.js" --include="*.ts" . 2>/dev/null | grep -v node_modules | grep -v test | grep -v comment

# Locate static analysis or linting configuration
find . -maxdepth 2 -name ".eslintrc*" -o -name "eslint.config.*" -o -name ".securityrc*" 2>/dev/null

# Execute security-focused linting rules if available
npm run lint:security 2>/dev/null || npm run lint 2>/dev/null || echo "(run linter manually)"
```

**Accept when:**
- All cryptographic hash operations use the crypto module's createHash API with SHA-256 algorithm explicitly specified
- No usage of deprecated or weak hash algorithms (MD5, SHA-1) exists in security-sensitive code paths
- Hash operations are centralized in core utility modules with test coverage demonstrating correct implementation
- Password hashing operations use dedicated algorithms (bcrypt, argon2) rather than SHA-256
- Code review process verifies no direct crypto module usage exists outside approved utility modules

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules R-CRYPTO-001 through R-CRYPTO-006 are mandatory for any code touching cryptographic hashing operations.
</enforcement>