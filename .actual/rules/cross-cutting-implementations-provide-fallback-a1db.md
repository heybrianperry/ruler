# Validate External JSON Configuration with JSON.parse Before Processing: Implementations Provide Fallback

These rules are ALWAYS ACTIVE for all agent configuration processing and MCP server integration workflows that consume external JSON files, including .mcp.json, firebender.json, and other user-editable JSON configuration files in nested directory structures.

### Rules

- **R-JSON-001** MUST: Wrap all `fs.readFile` + `JSON.parse` sequences for external configuration files in try-catch blocks with error logging that includes the file path and error message.
- **R-JSON-002** MUST: Use defensive property access patterns (e.g., `data.mcpServers ?? data.servers ?? {}`) when extracting configuration properties from parsed JSON to handle schema variations.
- **R-JSON-003** MAY: Implementations MAY provide fallback or default configurations when JSON parsing fails for non-critical configuration files, provided that parsing failures are always logged with clear file paths and error messages.
- **R-JSON-004** MUST: Never silently ignore JSON parsing failures; all failures must be logged at minimum with console.warn or equivalent, identifying the specific configuration file that failed.
- **R-JSON-005** SHOULD: Create and use a shared utility function (e.g., `loadJsonConfig(filePath, options)`) that encapsulates parsing, error handling, and optional schema validation to ensure consistent error handling patterns across all configuration consumers.

### Verify

```bash
# Check for unprotected JSON.parse calls on external files
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l | awk '{if ($1 > 0) exit 1}'

# Verify try-catch coverage around fs.readFile + JSON.parse sequences
grep -r 'fs\.readFile.*\.json' src/ | grep -A 5 'JSON\.parse' | grep -c 'catch'

# Run integration tests for nested MCP behavior
npm test -- --testNamePattern='nested-mcp-behavior' --verbose
```

**Accept when:**
- All JSON.parse calls on external configuration files are wrapped in try-catch blocks with error logging
- Integration tests for nested MCP propagation pass, demonstrating successful parsing of multi-level configuration hierarchies
- No unhandled JSON parsing exceptions occur during agent configuration loading in production logs
- All configuration loading failures are logged with file paths and error details
- Defensive property access patterns are used for schema-variable configuration properties

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All pull requests modifying configuration loading logic must pass the verify commands before merge. Production incidents caused by unhandled JSON parsing errors trigger immediate hotfix and post-mortem review.
</enforcement>