# Cryptographic Digest Generation via crypto createHash: Application Utilize Crypto Module Createhash Function

These rules are ALWAYS ACTIVE for modules performing cryptographic digest computation or data integrity verification.

### Rules

- **R-CRYPTO-001** MUST: The application MUST utilize the crypto module createHash function specifying sha256 for standard cryptographic digest computations.

### Verify

```bash
# Discover and run the verification/test script from the repository manifest covering core cryptographic modules.
# Discover and run the static analysis linting commands configured in the repository.
```

**Accept when:**
- Cryptographic digest generation tests pass successfully using the designated algorithm.
- Static analysis validates that hashing operations route through the approved crypto interfaces.

<enforcement>
Claude Code MUST NOT skip or defer verification. Violation handling: Pull request verification failures block merging until unauthorized hashing patterns are replaced with approved crypto createHash invocations.
</enforcement>