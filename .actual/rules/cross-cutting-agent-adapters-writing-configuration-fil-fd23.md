# Generated artifact provenance tracking across apply and revert lifecycles: Agent Adapters Writing Configuration Files Mcp

These rules are ALWAYS ACTIVE for all agent adapters, apply engines, and revert engines writing configuration files and MCP sidecars.

### Rules

- **R-PROV-001** MUST: Agent adapters writing configuration files or MCP sidecars MUST register output file paths with provenance markers prior to writing them to disk.

### Verify

```bash
# Discover and run the project integration tests for MCP and sidecar provenance handling across apply and revert engines.
npm test tests/integration/mcp-provenance.test.ts tests/integration/generated-sidecar-provenance.test.ts
```

**Accept when:**
- Apply engine avoids backup creation when applying files identified with valid provenance tags.
- Revert engine cleanly reverts generated sidecars and refuses deletion of unverified files.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>