# Use Node.js crypto Module for SHA-256 Hashing in Core Libraries: Hash Functions Exposed

These rules are ALWAYS ACTIVE for all core library modules requiring cryptographic hashing and public API contracts that expose hashing functionality.

### Rules

- **R-HASH-001** MUST: Hash functions exposed as public API contracts MUST include both sha256 and stableJson utilities.

### Verify

```bash
# Verify sha256 usage in core modules
grep -r "createHash('sha256')" src/core/ | grep -v node_modules

# Verify crypto module import
grep -r "from 'crypto'" src/core/hash.ts

# Verify public API exports
grep -E "export.*(sha256|stableJson)" src/core/hash.ts
```

**Accept when:**
- The core hash module imports createHash from Node.js crypto module
- SHA-256 is used via createHash('sha256') for cryptographic hashing operations
- Public API exports include both sha256 and stableJson functions as documented contracts

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting changes to core hash modules.
</enforcement>