# Hierarchical nested workspace configuration and root discovery: Config Resolution Revert Handlers Strictly Bound

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-CFG-001** MUST: Config resolution and revert handlers MUST strictly bound agent scoping to the nearest discovered workspace root unless explicit inheritance is defined.

### Verify

```bash
npx jest tests/integration/nested-config-cli.test.ts tests/integration/root-agents-detection.test.ts
```

**Accept when:**
- All nested config CLI and root agent detection integration tests pass successfully.
- Agent configuration scoping is strictly bound to the nearest discovered workspace root without implicit top-level leakage.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>