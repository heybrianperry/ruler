# Adoption of Internal Types Module for Centralized Domain Contracts: Before Integrating External Dependencies Typing Third

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-INT-001** MUST: Before integrating external dependencies or typing third-party structures, developers MUST locate the dependency manifest and lock artifact in the repository to verify exact version alignment.

### Verify

```bash
# Discover and run the workspace static type checker from project manifests
# Discover and run the project automated test suite
```

**Accept when:**
- All subsystem modules import shared contracts from the designated internal types module without local interface duplication.
- Static type analysis across the entire project repository completes with zero errors.

<enforcement>
Claude Code MUST NOT skip or defer verification. Compliance is verified by automated static type analysis in CI pipelines and peer code review.
</enforcement>