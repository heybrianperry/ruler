# Generated artifact provenance tracking across apply and revert lifecycles: Apply Engine Skip Backup Creation Files

These rules are ALWAYS ACTIVE for apply engine, revert engine, and agent adapters interacting with configuration files and MCP sidecars.

### Rules

- **R-PROV-001** MUST: The apply engine MUST skip backup creation for files tagged with verified generated artifact provenance markers.

### Verify

```bash
npm test tests/integration/mcp-provenance.test.ts tests/integration/generated-sidecar-provenance.test.ts
```

**Accept when:**
- Apply engine avoids backup creation when applying files identified with valid provenance tags.
- Revert engine cleanly reverts generated sidecars and refuses deletion of unverified files.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>