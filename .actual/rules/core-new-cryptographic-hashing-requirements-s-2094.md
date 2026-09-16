# Adoption of Node.js Crypto Module for SHA256 Hashing: New Cryptographic Hashing Requirements Sha256 Default

These rules are ALWAYS ACTIVE for code requiring cryptographic hashing for data protection.

### Rules

- SHOULD For new cryptographic hashing requirements, SHA256 SHOULD be the default algorithm unless specific security or performance requirements dictate an alternative.

### Verify

```bash
# Inspect the project's dependency manifest to confirm the presence of Node.js as a runtime dependency.
# Examine relevant source files for calls to cryptographic hashing functions.
# Run the project's test suite to ensure cryptographic operations are correctly implemented.
```

**Accept when:**
- The project's dependency manifest explicitly lists Node.js as a runtime.
- Source code demonstrates usage of the `crypto` module's `createHash('sha256')` function for hashing.
- All security-related tests pass without errors or warnings.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>