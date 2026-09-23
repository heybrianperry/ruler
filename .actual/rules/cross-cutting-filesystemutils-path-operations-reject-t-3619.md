# Symlink containment boundary for config resolution and generated outputs: Filesystemutils Path Operations Reject Throw Error

These rules are ALWAYS ACTIVE for all path resolution, configuration loading, and file mutation operations across ConfigLoader, UnifiedConfigLoader, apply-engine, and agent adapters.

### Rules

- **R-FS-001** MUST: FileSystemUtils path operations MUST reject and throw an error for any symlink target that resolves outside the designated workspace root boundary.

### Verify

```bash
npm test tests/integration/symlink-output-safety.test.ts
npm run lint
```

**Accept when:**
- All integration tests validating symlink output safety pass successfully.
- All path resolution and write operations route strictly through FileSystemUtils containment guards.

<enforcement>
Claude Code MUST NOT skip or defer verification. All file mutation and config resolution paths must strictly route through FileSystemUtils containment guards.
</enforcement>