# Adoption of Node.js Crypto Module for SHA256 Hashing: Cryptographic Hashing Operations Requiring Sha256 Utilize

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-SHA256-001** MUST: All cryptographic hashing operations requiring SHA256 MUST utilize the `createHash('sha256')` function provided by the `crypto` module.

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