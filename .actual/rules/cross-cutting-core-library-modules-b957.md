# Use Node.js crypto Module for SHA-256 Hashing in Core Libraries: Core Library Modules

These rules are ALWAYS ACTIVE for all core library modules requiring cryptographic hashing operations, public API contracts that expose hashing functionality, content addressing and integrity verification systems, and data structures requiring deterministic serialization before hashing.

### Rules

- **R-CRYPTO-001** MUST: Core library modules MUST use the Node.js crypto module for cryptographic hashing operations.
- **R-CRYPTO-002** MUST: Import createHash from the 'crypto' module and instantiate with 'sha256' algorithm parameter.
- **R-CRYPTO-003** MUST: Implement stableJson utility that sorts object keys recursively before JSON.stringify to ensure deterministic serialization.
- **R-CRYPTO-004** MUST: Export both sha256 and stableJson functions as public API contracts from core hash module.
- **R-CRYPTO-005** SHOULD: Implement streaming hash updates using hash.update() for large inputs to reduce memory pressure.
- **R-CRYPTO-006** SHOULD: Document stable serialization rules including handling of undefined values, functions, and non-serializable types.

### Verify

```bash
# Verify crypto module usage in core libraries
grep -r "createHash('sha256')" src/core/ | grep -v node_modules

# Verify crypto import in hash module
grep -r "from 'crypto'" src/core/hash.ts

# Verify public API exports
grep -E "export.*(sha256|stableJson)" src/core/hash.ts
```

**Accept when:**
- The core hash module imports createHash from Node.js crypto module
- SHA-256 is used via createHash('sha256') for cryptographic hashing operations
- Public API exports include both sha256 and stableJson functions as documented contracts
- No external cryptographic libraries (crypto-js, noble-hashes, etc.) are imported in core modules
- Deterministic hash output is validated across different inputs and execution contexts via unit tests

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for core library modules. Violations must be escalated to the security team and core library maintainers before merge.
</enforcement>