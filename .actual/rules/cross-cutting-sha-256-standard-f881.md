# Use Node.js crypto Module for SHA-256 Hashing in Core Libraries: Sha 256 Standard

These rules are ALWAYS ACTIVE for all core library modules requiring cryptographic hashing, public API contracts that expose hashing functionality, content addressing and integrity verification systems, and data structures requiring deterministic serialization before hashing.

### Rules

- **R-SHA256-001** MUST: SHA-256 MUST be the standard hashing algorithm for content addressing and integrity verification in core libraries.
- **R-SHA256-002** MUST: Core hash module MUST import `createHash` from the Node.js `'crypto'` module.
- **R-SHA256-003** MUST: SHA-256 hashing operations MUST use `createHash('sha256')` for cryptographic hashing.
- **R-SHA256-004** MUST: Public API MUST export both `sha256` and `stableJson` functions as documented contracts from the core hash module.
- **R-SHA256-005** MUST: Core modules MUST NOT import external cryptographic libraries (e.g., crypto-js, noble-hashes) without documented exception and security team approval.
- **R-SHA256-006** SHOULD: Implement `stableJson` utility that sorts object keys recursively before `JSON.stringify` to ensure deterministic serialization.
- **R-SHA256-007** SHOULD: Document stable serialization rules including handling of undefined values, functions, and non-serializable types.
- **R-SHA256-008** MAY: Consider implementing streaming hash updates using `hash.update()` for large inputs to reduce memory pressure.

### Verify

```bash
# Verify SHA-256 usage in core libraries
grep -r "createHash('sha256')" src/core/ | grep -v node_modules

# Verify crypto module import
grep -r "from 'crypto'" src/core/hash.ts

# Verify public API exports
grep -E "export.*(sha256|stableJson)" src/core/hash.ts
```

**Accept when:**
- The core hash module imports `createHash` from Node.js crypto module
- SHA-256 is used via `createHash('sha256')` for cryptographic hashing operations
- Public API exports include both `sha256` and `stableJson` functions as documented contracts
- No external cryptographic dependencies are present in core modules without documented exception
- Unit tests validate deterministic hash output across different inputs and execution contexts

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline MUST fail if core modules import external cryptographic libraries without documented exception. Code review MUST block merges that introduce non-standard hashing approaches in core libraries. Automated alerts MUST notify security team of cryptographic dependency changes.
</enforcement>