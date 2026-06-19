# Validate External JSON Configuration with JSON.parse Before Processing: Json Parse Operations

These rules are ALWAYS ACTIVE for all agent configuration processing and MCP server integration workflows that read external JSON configuration files from the filesystem.

### Rules

- **R-JSON-001** MUST: JSON.parse() operations MUST be wrapped in try-catch blocks to handle syntax errors and invalid JSON gracefully.
- **R-JSON-002** MUST: All fs.readFile() operations for JSON configuration files (.mcp.json, firebender.json) must include error handling for parse failures.
- **R-JSON-003** SHOULD: Use console.warn() or structured logging to record JSON.parse() failures with file path and error details.
- **R-JSON-004** SHOULD: After successful JSON.parse(), use optional chaining (?.) or explicit property checks before accessing nested configuration properties.
- **R-JSON-005** MAY: Extract a shared parseJsonConfig(filePath: string) utility function to centralize error handling and logging logic across agents and configuration modules.

### Verify

```bash
# Check for unprotected JSON.parse() calls in configuration code
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l | grep '^0$'

# Check for fs.readFile operations on JSON files without JSON.parse protection
grep -r 'fs\.readFile.*\.json' src/ | grep -L 'JSON\.parse'

# Run integration tests for nested MCP configuration parsing
npm test -- --testNamePattern='Nested MCP propagation' --verbose
```

**Accept when:**
- All JSON.parse() invocations for configuration files are wrapped in try-catch blocks with error logging
- Integration tests for nested MCP configuration successfully parse JSON from multiple directory levels without crashes
- Grep verification shows no unprotected JSON.parse() calls in configuration loading code paths
- Configuration files from project root, module, and submodule directories are validated before property access

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse() operations reading external configuration files MUST be wrapped in try-catch blocks. Violations block CI pipeline and code review.
</enforcement>