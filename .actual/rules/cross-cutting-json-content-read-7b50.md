# Validate External JSON Configuration with JSON.parse Before Processing: Json Content Read

These rules are ALWAYS ACTIVE for all agent configuration processing and MCP server integration workflows that read JSON configuration files from the filesystem.

### Rules

- **R-JSON-001** MUST: All JSON content read from filesystem configuration files MUST be parsed using JSON.parse() before accessing properties or performing operations on the data structure.

### Verify

```bash
# Check for unprotected JSON.parse() calls without try-catch
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l | grep '^0$'

# Check for fs.readFile operations on JSON files without JSON.parse
grep -r 'fs\.readFile.*\.json' src/ | grep -L 'JSON\.parse'

# Run integration tests for nested MCP configuration parsing
npm test -- --testNamePattern='Nested MCP propagation' --verbose
```

**Accept when:**
- All JSON.parse() invocations for configuration files are wrapped in try-catch blocks with error logging
- Integration tests for nested MCP configuration successfully parse JSON from multiple directory levels without crashes
- Grep verification shows no unprotected JSON.parse() calls in configuration loading code paths
- All MCP configuration files (.mcp.json) read from project root, module, and submodule directories are validated before property access
- Agent-specific configuration files (firebender.json) loaded from filesystem are validated before property access
- Any JSON configuration file processed by applyRulerConfig, loadExistingConfig, or applyAllAgentConfigs functions is validated before property access

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON content read from filesystem configuration files MUST be validated with JSON.parse() in try-catch blocks before accessing properties. Integration tests MUST pass without unhandled JSON parse errors. Code review MUST reject pull requests that read JSON configuration files without proper error handling.
</enforcement>
