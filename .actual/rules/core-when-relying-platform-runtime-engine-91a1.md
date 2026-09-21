# Cryptographic Digest Generation via crypto createHash: When Relying Platform Runtime Engine Dependencies

These rules are ALWAYS ACTIVE for modules performing cryptographic digest computation or data integrity verification.

### Rules

- **R-CRYPTO-001** MUST: When relying on platform runtime or engine dependencies, the consumer MUST inspect the environment resolution artifacts to ensure required cryptographic interfaces are fully supported in the target deployment target.

### Verify

```bash
# Discover and run the project verification script from the repository manifest
# Discover and run static analysis linting commands configured in the repository
```

**Accept when:**
- Cryptographic digest generation tests pass successfully using the designated algorithm.
- Static analysis validates that hashing operations route through the approved crypto interfaces.

<enforcement>
Claude Code MUST NOT skip or defer verification. Verification is mandatory.
</enforcement>