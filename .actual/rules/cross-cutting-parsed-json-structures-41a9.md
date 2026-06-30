# Validate External JSON Configuration with JSON.parse Before Processing: Parsed Json Structures

These rules are ALWAYS ACTIVE for all agent configuration processing and MCP server integration workflows that consume external JSON files, including .mcp.json, firebender.json, and other user-editable JSON configuration files in nested directory structures.

### Rules

- **R-JSON-001** SHOULD: Parsed JSON structures SHOULD be type-checked or validated against expected schemas before extracting nested properties.

### Verify

```bash
# Check for unprotected JSON.parse calls on external configuration files
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l | awk '{if ($1 > 0) exit 1}'

# Verify JSON.parse calls on fs.readFile results are wrapped in catch blocks
grep -r 'fs\.readFile.*\.json' src/ | grep -A 5 'JSON\.parse' | grep -c 'catch'

# Run nested MCP behavior integration tests
npm test -- --testNamePattern='nested-mcp-behavior' --verbose
```

**Accept when:**
- All JSON.parse calls on external configuration files are wrapped in try-catch blocks with error logging
- Integration tests for nested MCP propagation pass, demonstrating successful parsing of multi-level configuration hierarchies
- No unhandled JSON parsing exceptions occur during agent configuration loading in production logs
- Defensive property access patterns (e.g., `data.mcpServers ?? data.servers ?? {}`) are used to handle schema variations

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON parsing in configuration loading workflows. All external JSON files MUST be validated before use. Pull requests with unprotected JSON.parse on external files must be revised before merge.
</enforcement>