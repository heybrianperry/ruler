# Validate External JSON Configuration with JSON.parse Before Processing: System Implement Schema

These rules are ALWAYS ACTIVE for all agent configuration processing and MCP server integration workflows, including all MCP configuration files (.mcp.json) read from project root, module, and submodule directories, agent-specific configuration files (firebender.json) loaded from filesystem, and any JSON configuration file processed by applyRulerConfig, loadExistingConfig, or applyAllAgentConfigs functions.

### Rules

- **R-JSON-001** MUST: Wrap all fs.readFile() operations for JSON configuration files with JSON.parse() in try-catch blocks, following the pattern established in FirebenderAgent.ts.
- **R-JSON-002** MUST: Use console.warn() or a structured logging framework to record parse failures with file path and error details.
- **R-JSON-003** MUST: After successful JSON.parse(), use optional chaining (?.) or explicit property checks before accessing nested configuration properties.
- **R-JSON-004** MAY: The system MAY implement schema validation beyond JSON.parse() for complex configuration structures with multiple optional properties.
- **R-JSON-005** SHOULD: Consider extracting a shared parseJsonConfig(filePath: string) utility function to centralize error handling and logging logic across agents and configuration modules.

### Verify

```bash
# Check for unprotected JSON.parse() calls in configuration loading code
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l | grep '^0$'

# Find fs.readFile operations for JSON files without JSON.parse protection
grep -r 'fs\.readFile.*\.json' src/ | grep -L 'JSON\.parse'

# Run nested MCP propagation integration tests
npm test -- --testNamePattern='Nested MCP propagation' --verbose
```

**Accept when:**
- All JSON.parse() invocations for configuration files are wrapped in try-catch blocks with error logging
- Integration tests for nested MCP configuration successfully parse JSON from multiple directory levels without crashes
- Grep verification shows no unprotected JSON.parse() calls in configuration loading code paths
- All fs.readFile() operations for .json configuration files include corresponding JSON.parse() with error handling

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON configuration file reads MUST be validated with JSON.parse() in try-catch blocks before property access. Integration tests MUST pass without unhandled parse errors.
</enforcement>