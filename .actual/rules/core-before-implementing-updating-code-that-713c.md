# Adoption of Node.js Crypto Module for SHA256 Hashing: Before Implementing Updating Code That Uses

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-CRYPTO-001** MUST: Before implementing or updating code that uses the `crypto` module, developers MUST discover the ecosystem's lock file and resolve the exact locked version of Node.js and its built-in modules to ensure API compatibility and security patch levels.

### Verify

```bash
# Inspect the project's dependency manifest to confirm the presence of Node.js as a runtime dependency.
# Examine relevant source files for calls to cryptographic hashing functions (e.g., crypto.createHash('sha256')).
# Run the project's test suite to ensure cryptographic operations are correctly implemented.
```

**Accept when:**
- The project's dependency manifest explicitly lists Node.js as a runtime.
- Source code demonstrates usage of the `crypto` module's `createHash('sha256')` function for hashing.
- All security-related tests pass without errors or warnings.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>