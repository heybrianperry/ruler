# Validate External JSON Configuration with JSON.parse Before Processing: External Json Configuration

These rules are ALWAYS ACTIVE for all agent configuration processing and MCP server integration workflows that consume external JSON files, including .mcp.json, firebender.json, and other user-editable JSON configuration files in nested directory structures.

### Rules

- **R-EJSON-001** MUST: All external JSON configuration files MUST be parsed using JSON.parse with explicit error handling (try-catch blocks) before accessing configuration properties.
- **R-EJSON-002** MUST: Wrap all fs.readFile + JSON.parse sequences in try-catch blocks, logging the file path and error message on failure.
- **R-EJSON-003** SHOULD: Use defensive property access patterns (e.g., `data.mcpServers ?? data.servers ?? {}`) to handle schema variations in parsed JSON structures.
- **R-EJSON-004** SHOULD: Consider creating shared utility functions for configuration loading that encapsulate parsing, error handling, and optional schema validation.

### Verify

```bash
# Check for unprotected JSON.parse calls on external files
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l | awk '{if ($1 > 0) exit 1}'

# Verify fs.readFile + JSON.parse sequences have catch handlers
grep -r 'fs\.readFile.*\.json' src/ | grep -A 5 'JSON\.parse' | grep -c 'catch'

# Run nested MCP behavior integration tests
npm test -- --testNamePattern='nested-mcp-behavior' --verbose
```

**Accept when:**
- All JSON.parse calls on external configuration files are wrapped in try-catch blocks with error logging
- Integration tests for nested MCP propagation pass, demonstrating successful parsing of multi-level configuration hierarchies
- No unhandled JSON parsing exceptions occur during agent configuration loading in production logs
- Defensive property access patterns are used for schema-variable configuration structures

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All external JSON configuration parsing MUST include explicit error handling before merge.
</enforcement>