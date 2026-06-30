# Validate External JSON Configuration with JSON.parse Before Processing: Configuration Processing Not

These rules are ALWAYS ACTIVE for all agent configuration processing and MCP server integration workflows that consume external JSON files, including MCP server configuration files (.mcp.json), agent-specific configuration files (firebender.json, copilot config, windsurf config), and user-editable JSON configuration consumed by applyAllAgentConfigs and related integration workflows.

### Rules

- **R-CONFIG-001** MUST: Configuration processing MUST NOT proceed with unparsed or partially parsed JSON content.
- **R-CONFIG-002** MUST: Wrap all fs.readFile + JSON.parse sequences in try-catch blocks, logging the file path and error message on failure.
- **R-CONFIG-003** MUST: Use defensive property access patterns (e.g., `data.mcpServers ?? data.servers ?? {}`) to handle schema variations in MCP server configurations.
- **R-CONFIG-004** SHOULD: Create and use a shared loadJsonConfig(filePath, options) utility that encapsulates parsing, error handling, and optional schema validation.
- **R-CONFIG-005** SHOULD: Document expected JSON schemas in comments or separate schema files to guide validation implementation and user configuration.

### Verify

```bash
# Check for unprotected JSON.parse calls on external configuration files
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l | awk '{if ($1 > 0) exit 1}'

# Verify try-catch wrapping around fs.readFile + JSON.parse sequences
grep -r 'fs\.readFile.*\.json' src/ | grep -A 5 'JSON\.parse' | grep -c 'catch'

# Run nested MCP behavior integration tests
npm test -- --testNamePattern='nested-mcp-behavior' --verbose
```

**Accept when:**
- All JSON.parse calls on external configuration files are wrapped in try-catch blocks with error logging
- Integration tests for nested MCP propagation pass, demonstrating successful parsing of multi-level configuration hierarchies
- No unhandled JSON parsing exceptions occur during agent configuration loading in production logs
- Defensive property access patterns are used for schema variation handling in MCP configurations

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse calls on external configuration files must be protected with try-catch blocks before code review approval.
</enforcement>