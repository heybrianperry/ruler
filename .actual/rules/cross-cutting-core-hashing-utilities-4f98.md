# Use Node.js crypto Module for SHA-256 Hashing in Core Libraries: Core Hashing Utilities

These rules are ALWAYS ACTIVE for all core library modules requiring cryptographic hashing, public API contracts that expose hashing functionality, content addressing and integrity verification systems, and data structures requiring deterministic serialization before hashing.

### Rules

- **R-CORE-HASH-001** MUST_NOT: Core hashing utilities MUST NOT depend on external cryptographic libraries when Node.js crypto provides equivalent functionality.
- **R-CORE-HASH-002** MUST: Import createHash from the 'crypto' module and instantiate with 'sha256' algorithm parameter for cryptographic hashing operations.
- **R-CORE-HASH-003** MUST: Implement stableJson utility that sorts object keys recursively before JSON.stringify to ensure deterministic serialization.
- **R-CORE-HASH-004** MUST: Export both sha256 and stableJson functions as public API contracts from the core hash module.
- **R-CORE-HASH-005** SHOULD: Implement streaming hash updates using hash.update() for large inputs to reduce memory pressure.
- **R-CORE-HASH-006** SHOULD: Document stable serialization rules including handling of undefined values, functions, and non-serializable types.

### Verify

```bash
# Verify Node.js crypto module usage in core libraries
grep -r "createHash('sha256')" src/core/ | grep -v node_modules

# Verify crypto module import in core hash module
grep -r "from 'crypto'" src/core/hash.ts

# Verify public API exports
grep -E "export.*(sha256|stableJson)" src/core/hash.ts
```

**Accept when:**
- The core hash module imports createHash from Node.js crypto module
- SHA-256 is used via createHash('sha256') for cryptographic hashing operations
- Public API exports include both sha256 and stableJson functions as documented contracts
- No external cryptographic libraries are imported in core hashing utilities
- Stable JSON serialization is implemented with recursive key sorting

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for core library code. Violations must be escalated to the security team and core library maintainers before merge.
</enforcement>