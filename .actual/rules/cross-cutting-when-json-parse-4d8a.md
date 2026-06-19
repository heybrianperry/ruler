# Validate External JSON Configuration with JSON.parse Before Processing: When Json Parse

These rules are ALWAYS ACTIVE for all agent configuration processing and MCP server integration workflows that read external JSON configuration files from the filesystem.

### Rules

- **R-JSON-001** MUST: When JSON.parse() fails, the system MUST log the error with context (file path, error message) and continue execution with fallback behavior.

### Verify

```bash
# Check for unprotected JSON.parse() calls in configuration loading code
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l | grep '^0$'

# Verify all fs.readFile operations for JSON files have JSON.parse protection
grep -r 'fs\.readFile.*\.json' src/ | grep -L 'JSON\.parse'

# Run integration tests for nested MCP configuration parsing
npm test -- --testNamePattern='Nested MCP propagation' --verbose
```

**Accept when:**
- All JSON.parse() invocations for configuration files are wrapped in try-catch blocks with error logging
- Integration tests for nested MCP configuration successfully parse JSON from multiple directory levels without crashes
- Grep verification shows no unprotected JSON.parse() calls in configuration loading code paths
- All MCP configuration files (.mcp.json) and agent-specific files (firebender.json) are validated before property access

<enforcement>
Claude Code MUST NOT skip or defer verification of JSON.parse() error handling in configuration loading paths. All external JSON configuration files MUST be wrapped in try-catch blocks with contextual error logging before any property access or business logic operations.
</enforcement>