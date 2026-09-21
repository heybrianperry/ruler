# Generated artifact provenance tracking and revert synchronization: Cleanup Revert Engines Refuse Deletion Files

These rules are ALWAYS ACTIVE for all source files, apply engines, revert engines, and cleanup routines managing generated artifacts and MCP sidecars.

### Rules

- **R-PROV-001** MUST: Cleanup and revert engines MUST refuse deletion of files lacking verifiable Ruler provenance tags.

### Verify

```bash
npm test tests/unit/core/apply-engine.test.ts tests/integration/mcp-provenance.test.ts tests/integration/generated-sidecar-provenance.test.ts
```

**Accept when:**
- Cleanup and revert engines successfully verify Ruler provenance tags prior to file deletion
- Files lacking verifiable provenance tags are rejected by cleanup routines and preserved
- Unit and integration tests for apply-engine, mcp-provenance, and generated-sidecar-provenance pass successfully

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>