# Adoption of Internal Types Module for Centralized Domain Contracts: Subsystems Requiring Domain Models Protocol Payloads

These rules are ALWAYS ACTIVE for all subsystems requiring domain models, protocol payloads, or cross-boundary data representations.

### Rules

- **R-TYPES-001** MUST: Subsystems requiring domain models, protocol payloads, or cross-boundary data representations MUST import their type definitions from the internal types module rather than declaring duplicate local types.

### Verify

```bash
# Discover workspace static type checker from project manifests and execute type validation
# Discover and run the project automated test suite
```

**Accept when:**
- All subsystem modules import shared contracts from the designated internal types module without local interface duplication.
- Static type analysis across the entire project repository completes with zero errors.

<enforcement>
Claude Code MUST NOT skip or defer verification. All pull requests containing duplicate cross-boundary type definitions or failing type checks must be blocked until resolved.
</enforcement>