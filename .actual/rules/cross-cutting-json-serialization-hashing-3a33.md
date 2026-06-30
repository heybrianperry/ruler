# Use Node.js crypto Module for SHA-256 Hashing in Core Libraries: Json Serialization Hashing

These rules are ALWAYS ACTIVE for all core library modules requiring cryptographic hashing, public API contracts that expose hashing functionality, content addressing and integrity verification systems, and data structures requiring deterministic serialization before hashing.

### Rules

- **R-CRYPTO-001** MUST: JSON serialization for hashing MUST use stable ordering to ensure deterministic hash outputs.
- **R-CRYPTO-002** MUST: Import `createHash` from the Node.js `'crypto'` module for SHA-256 hashing operations in core libraries.
- **R-CRYPTO-003** MUST: Use `createHash('sha256')` for all cryptographic hashing operations in core hash modules.
- **R-CRYPTO-004** MUST: Export both `sha256` and `stableJson` functions as public API contracts from the core hash module.
- **R-CRYPTO-005** MUST: Implement `stableJson` utility that sorts object keys recursively before `JSON.stringify` to ensure deterministic serialization.
- **R-CRYPTO-006** SHOULD: Document stable serialization rules including handling of undefined values, functions, and non-serializable types.
- **R-CRYPTO-007** SHOULD: Consider implementing streaming hash updates using `hash.update()` for large inputs to reduce memory pressure.

### Verify

```bash
# Verify crypto module usage in core libraries
grep -r "createHash('sha256')" src/core/ | grep -v node_modules

# Verify crypto import statement
grep -r "from 'crypto'" src/core/hash.ts

# Verify public API exports
grep -E "export.*(sha256|stableJson)" src/core/hash.ts
```

**Accept when:**
- The core hash module imports `createHash` from Node.js crypto module
- SHA-256 is used via `createHash('sha256')` for cryptographic hashing operations
- Public API exports include both `sha256` and `stableJson` functions as documented contracts
- Stable JSON serialization is implemented with recursive key sorting
- All core library modules use the exported hash functions rather than external cryptographic libraries

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All R-CRYPTO rules must be validated before accepting changes to core hash modules or cryptographic operations.
</enforcement>