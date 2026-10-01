# Adoption of Internal Types Module for Centralized Domain Contracts: Shared Domain Contracts Exposed Internal Types

These rules are ALWAYS ACTIVE for all domain models, data transfer objects, configuration interfaces, and protocol definitions shared across subsystems, including editor integrations, protocol adapters, and core execution layers.

### Rules

- **R-DOMAIN-001** SHOULD: Shared domain contracts exposed by the internal types module maintain backward compatibility when fields are introduced or modified.

### Verify

```bash
# Discover the workspace static type checker from project manifests and execute type validation
# Discover and run the project automated test suite to ensure contract compatibility
```

**Accept when:**
- All subsystem modules import shared contracts from the designated internal types module without local interface duplication.
- Static type analysis across the entire project repository completes with zero errors.

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated static type analysis and peer review must enforce centralized contract imports.
</enforcement>