# Validate External JSON Configuration with JSON.parse Before Processing: Type Assertions Runtime

These rules are ALWAYS ACTIVE for all agent configuration processing and MCP server integration workflows, including all MCP configuration files (.mcp.json) read from project root, module, and submodule directories, agent-specific configuration files (firebender.json) loaded from filesystem, and any JSON configuration file processed by applyRulerConfig, loadExistingConfig, or applyAllAgentConfigs functions.

### Rules

- **R-JSON-001** SHOULD: Type assertions or runtime type checks SHOULD be applied to parsed JSON objects to ensure they conform to expected Record<string, unknown> or similar structures before accessing properties or performing business logic operations.

### Verify

```bash
# Check for unprotected JSON.parse() calls in configuration loading code
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l | grep '^0$'

# Identify fs.readFile operations for JSON files without JSON.parse protection
grep -r 'fs\.readFile.*\.json' src/ | grep -L 'JSON\.parse'

# Run integration tests for nested MCP configuration parsing
npm test -- --testNamePattern='Nested MCP propagation' --verbose
```

**Accept when:**
- All JSON.parse() invocations for configuration files are wrapped in try-catch blocks with error logging
- Integration tests for nested MCP configuration successfully parse JSON from multiple directory levels without crashes
- Grep verification shows no unprotected JSON.parse() calls in configuration loading code paths
- All fs.readFile() operations for JSON configuration files include JSON.parse() with error handling
- Configuration parsing failures are logged with file path and error details using console.warn() or structured logging

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse() invocations for external configuration files MUST be wrapped in try-catch blocks with appropriate error logging and graceful degradation. Code review MUST reject pull requests that read JSON configuration files without proper error handling. CI pipeline MUST fail if integration tests detect unhandled JSON parse errors.
</enforcement>