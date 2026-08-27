# Use Node.js crypto Module with SHA-256 for Cryptographic Hashing: Hash Functionality Centralized Core Utility Modules

These rules are ALWAYS ACTIVE for all files implementing cryptographic hashing operations, core utility modules providing hash operations, security-sensitive data processing, content addressing, and data integrity verification use cases.

### Rules

- **R-HASH-001** SHOULD: Hash functionality SHOULD be centralized in core utility modules rather than implemented inline across the codebase.
- **R-HASH-002** MUST: All cryptographic hash operations use the crypto module's createHash API with SHA-256 algorithm specification.
- **R-HASH-003** MUST: No usage of deprecated or weak hash algorithms (MD5, SHA-1) exists in security-sensitive code paths.
- **R-HASH-004** MUST: Hash operations are centralized in core utility modules with test coverage demonstrating correct implementation.
- **R-HASH-005** SHOULD: When implementing hash operations, always specify the algorithm explicitly in createHash calls rather than relying on defaults.
- **R-HASH-006** SHOULD: For data integrity use cases, consider whether HMAC (keyed hashing) provides better security properties than plain hashing for the specific threat model.

### Verify

```bash
# Discover the project's static analysis or linting configuration and execute the security-focused rules to verify crypto module usage patterns
# (Exact command depends on project's linting tool — check project configuration)

# Locate the project's test suite and execute tests covering the core hash utility module to verify SHA-256 implementation correctness
# (Exact command depends on project's test runner — check package.json or build configuration)

# Search the codebase for all hash-related operations and verify they route through centralized utilities rather than inline crypto module usage
grep -r "require.*crypto" --include="*.js" --include="*.ts" | grep -v "node_modules" | grep -v "test"
grep -r "createHash" --include="*.js" --include="*.ts" | grep -v "node_modules" | grep -v "test"
grep -r "md5\|MD5\|sha1\|SHA-1\|sha-1" --include="*.js" --include="*.ts" | grep -v "node_modules" | grep -v "test"
```

**Accept when:**
- All cryptographic hash operations use the crypto module's createHash API with SHA-256 algorithm specification
- No usage of deprecated or weak hash algorithms (MD5, SHA-1) exists in security-sensitive code paths
- Hash operations are centralized in core utility modules with test coverage demonstrating correct implementation
- Direct crypto module usage outside approved utility modules is not present or has been reviewed and approved
- The Node.js version in use supports the required crypto APIs as verified against the project's runtime version specification

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules marked MUST are mandatory; violations identified in code review must be refactored to use centralized hash utilities before merge. Rules marked SHOULD represent strong architectural preferences and should be followed unless documented exceptions are approved by the security team.
</enforcement>