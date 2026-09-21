# Cryptographic Digest Generation via crypto createHash: Components Not Instantiate Custom Unverified Hashing

These rules are ALWAYS ACTIVE for modules performing cryptographic digest computation or data integrity verification.

### Rules

- **R-CRYPTO-001** MUST_NOT: Components MUST NOT instantiate custom or unverified hashing implementations when cryptographic digest generation is required.

### Verify

```bash
# Discover the project verification script from the repository manifest and execute the test suite covering core cryptographic modules.
# Discover and run the static analysis linting commands configured in the repository to verify that direct hash instantiation conforms to approved rules.
```

**Accept when:**
- Cryptographic digest generation tests pass successfully using the designated algorithm.
- Static analysis validates that hashing operations route through the approved crypto interfaces.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>