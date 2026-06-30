# Use Node.js crypto Module for SHA-256 Hashing in Core Libraries: Hash Implementations Use

These rules are ALWAYS ACTIVE for all core library modules requiring cryptographic hashing, public API contracts that expose hashing functionality, content addressing and integrity verification systems, and data structures requiring deterministic serialization before hashing.

### Rules

- **R-HASH-001** SHOULD: Hash implementations SHOULD use createHash('sha256') from the crypto module rather than third-party libraries.

### Verify

```bash
# Verify SHA-256 usage in core library modules
grep -r "createHash('sha256')" src/core/ | grep -v node_modules

# Verify crypto module import in hash module
grep -r "from 'crypto'" src/core/hash.ts

# Verify public API exports
grep -E "export.*(sha256|stableJson)" src/core/hash.ts
```

**Accept when:**
- The core hash module imports createHash from Node.js crypto module
- SHA-256 is used via createHash('sha256') for cryptographic hashing operations
- Public API exports include both sha256 and stableJson functions as documented contracts
- Stable JSON serialization is implemented with recursive key sorting for deterministic output
- No external cryptographic libraries (crypto-js, noble-hashes, etc.) are imported in core modules

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated code review checks for crypto module usage in core libraries are mandatory. Static analysis tools MUST scan for third-party cryptographic dependencies. Unit tests MUST validate deterministic hash output across different inputs and execution contexts.
</enforcement>