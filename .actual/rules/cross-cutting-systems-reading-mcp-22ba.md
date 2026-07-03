# Validate External JSON Configuration with JSON.parse Before Processing: Systems Reading Mcp

These rules are ALWAYS ACTIVE for all agent configuration processing and MCP server integration workflows that consume external JSON files, including .mcp.json, firebender.json, and other user-editable JSON configuration files in nested directory structures.

### Rules

- **R-MCP-001** MUST: Wrap all `fs.readFile` + `JSON.parse` sequences for external configuration files in try-catch blocks, logging the file path and error message on failure.
- **R-MCP-002** SHOULD: Handle schema variations in MCP server configurations through defensive property access patterns (e.g., `data.mcpServers ?? data.servers ?? {}`).
- **R-MCP-003** SHOULD: Create and use shared utility functions for configuration loading that encapsulate parsing, error handling, and optional schema validation.
- **R-MCP-004** MUST: Always log parsing failures with clear file paths and error messages to prevent silent configuration failures.

### Verify

```bash
# Check for unprotected JSON.parse calls on external files
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l | awk '{if ($1 > 0) exit 1}'

# Verify fs.readFile + JSON.parse sequences have catch blocks
grep -r 'fs\.readFile.*\.json' src/ | grep -A 5 'JSON\.parse' | grep -c 'catch'

# Run nested MCP behavior integration tests
npm test -- --testNamePattern='nested-mcp-behavior' --verbose
```

**Accept when:**
- All JSON.parse calls on external configuration files are wrapped in try-catch blocks with error logging
- Integration tests for nested MCP propagation pass, demonstrating successful parsing of multi-level configuration hierarchies
- No unhandled JSON parsing exceptions occur during agent configuration loading in production logs
- Defensive property access patterns are used for schema variations (mcpServers vs servers)

<enforcement>
Claude Code MUST NOT skip or defer verification. All pull requests with unprotected JSON.parse on external files must be revised before merge. Production incidents caused by unhandled JSON parsing errors trigger immediate hotfix and post-mortem review.
</enforcement>