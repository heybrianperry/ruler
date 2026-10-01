# Adoption of Internal Types Module for Centralized Domain Contracts: Subsystems Define Private Helper Types Scoped

These rules are ALWAYS ACTIVE for domain models, data transfer objects, configuration interfaces, and protocol definitions shared across subsystems, including editor integrations, protocol adapters, and core execution layers.

### Rules

- **R-INT-001** MAY: Subsystems MAY define private helper types scoped strictly to internal module implementation details if those types never cross module boundaries.

### Verify

```bash
# Discover the workspace static type checker from project manifests and execute the project-wide type validation script.
# Discover and run the project automated test suite to ensure contract compatibility.
```

**Accept when:**
- All subsystem modules import shared contracts from the designated internal types module without local interface duplication.
- Static type analysis across the entire project repository completes with zero errors.

<enforcement>
Claude Code MUST NOT skip or defer verification. All pull requests containing duplicate cross-boundary type definitions or failing type checks must be blocked until resolved.
</enforcement>