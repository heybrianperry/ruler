# Generated artifact provenance tracking and revert synchronization: Files Created Updated Apply Engine Record

These rules are ALWAYS ACTIVE for all files created, updated, or managed by the apply-engine and revert-engine.

### Rules

- **R-PROV-001** MUST: All files created or updated by apply-engine MUST record provenance metadata.

### Verify

```bash
npx jest tests/unit/core/apply-engine.test.ts tests/integration/mcp-provenance.test.ts tests/integration/generated-sidecar-provenance.test.ts
```

**Accept when:**
- All apply-engine, revert-engine, and provenance integration tests pass successfully.
- Every file generated or updated by the apply-engine contains verifiable provenance tags.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>