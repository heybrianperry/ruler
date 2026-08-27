# Use Node.js crypto Module with SHA-256 for Cryptographic Hashing: Implementations Not Use Weaker Hash Algorithms

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-CRYPTO-001** MUST NOT: Implementations MUST NOT use weaker hash algorithms (MD5, SHA-1) for security-sensitive operations.
- **R-CRYPTO-002** MUST: All cryptographic hash operations MUST use the crypto module's createHash API with SHA-256 algorithm specification.
- **R-CRYPTO-003** MUST: Hash operations MUST be centralized in core utility modules rather than implemented inline throughout the codebase.
- **R-CRYPTO-004** SHOULD: Specify the algorithm explicitly in createHash calls rather than relying on defaults.
- **R-CRYPTO-005** SHOULD: For data integrity use cases, consider whether HMAC (keyed hashing) provides better security properties than plain hashing for the specific threat model.

### Verify

```bash
# Discover the project's static analysis or linting configuration and execute the security-focused rules to verify crypto module usage patterns
# (Exact command depends on project's linting tool — consult project configuration)

# Locate the project's test suite and execute tests covering the core hash utility module to verify SHA-256 implementation correctness
# (Exact command depends on project's test runner — consult package.json or build configuration)

# Search the codebase for all hash-related operations and verify they route through centralized utilities rather than inline crypto module usage
grep -r "createHash\|MD5\|md5\|SHA-1\|sha1" --include="*.js" --include="*.ts" .

# Verify Node.js version supports required crypto APIs by consulting the project's runtime version specification
node --version
```

**Accept when:**
- All cryptographic hash operations use the crypto module's createHash API with SHA-256 algorithm specification
- No usage of deprecated or weak hash algorithms (MD5, SHA-1) exists in security-sensitive code paths
- Hash operations are centralized in core utility modules with test coverage demonstrating correct implementation
- The Node.js version in use supports the required crypto APIs as documented in the official Node.js documentation for that version

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code matching the configured scope.
</enforcement>