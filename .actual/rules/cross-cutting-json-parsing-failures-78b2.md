# Validate External JSON Configuration with JSON.parse Before Processing: Json Parsing Failures

These rules are ALWAYS ACTIVE for all agent configuration processing and MCP server integration workflows that consume external JSON files, including .mcp.json, firebender.json, and other user-editable JSON configuration files in nested directory structures.

### Rules

- **R-JSON-001** MUST: JSON parsing failures MUST be caught and logged with context identifying the file path and error details.

### Verify

```bash
# Detect unprotected JSON.parse calls on external configuration files
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l | awk '{if ($1 > 0) exit 1}'

# Verify fs.readFile + JSON.parse sequences include catch blocks
grep -r 'fs\.readFile.*\.json' src/ | grep -A 5 'JSON\.parse' | grep -c 'catch'

# Run nested MCP behavior integration tests
npm test -- --testNamePattern='nested-mcp-behavior' --verbose
```

**Accept when:**
- All JSON.parse calls on external configuration files are wrapped in try-catch blocks with error logging
- Integration tests for nested MCP propagation pass, demonstrating successful parsing of multi-level configuration hierarchies
- No unhandled JSON parsing exceptions occur during agent configuration loading in production logs

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse calls on external configuration files must be protected with try-catch blocks and logged with file path context before code is committed.
</enforcement>