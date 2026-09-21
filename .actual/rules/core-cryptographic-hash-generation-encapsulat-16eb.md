# Cryptographic Digest Generation via crypto createHash: Cryptographic Hash Generation Encapsulated Within Dedicated

These rules are ALWAYS ACTIVE for modules performing cryptographic digest computation or data integrity verification.

### Rules

- **R-CRYPTO-001** SHOULD: Cryptographic hash generation SHOULD be encapsulated within dedicated core utility interfaces rather than distributed across domain components.

### Verify

```bash
# Discover and run the project verification script from the repository manifest covering core cryptographic modules
# Discover and run static analysis linting commands configured in the repository to verify direct hash instantiation conforms to approved rules
```

**Accept when:**
- Cryptographic digest generation tests pass successfully using the designated algorithm.
- Static analysis validates that hashing operations route through the approved crypto interfaces.

<enforcement>
Claude Code MUST NOT skip or defer verification. Violation handling: Pull request verification failures block merging until unauthorized hashing patterns are replaced with approved crypto createHash invocations.
</enforcement>