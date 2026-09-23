# Generated artifact provenance tracking across apply and revert lifecycles: Revert Engine Not Delete Overwrite Target

These rules are ALWAYS ACTIVE for all source files, apply/revert engine implementations, and agent adapters writing configuration sidecars and MCP configurations.

### Rules

- **R-PROV-001** MUST: The revert engine MUST NOT delete or overwrite target files that lack verified tool-generated provenance markers.

### Verify

```bash
npm test tests/integration/mcp-provenance.test.ts tests/integration/generated-sidecar-provenance.test.ts
```

**Accept when:**
- Apply engine avoids backup creation when applying files identified with valid provenance tags.
- Revert engine cleanly reverts generated sidecars and refuses deletion of unverified files.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pull request checks fail if adapter sidecars are written without registering provenance, and apply/revert lifecycle runs abort or fail safety checks when file ownership cannot be validated.
</enforcement>