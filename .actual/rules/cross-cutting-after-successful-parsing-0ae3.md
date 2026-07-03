# Validate External JSON Configuration with JSON.parse Before Processing: After Successful Parsing

These rules are ALWAYS ACTIVE for all agent configuration processing and MCP server integration workflows that read external JSON configuration files from the filesystem.

### Rules

- **R-JSON-001** SHOULD: After successful parsing, the system SHOULD validate the presence of expected top-level properties (e.g., mcpServers, servers) before accessing nested data

### Verify

```bash
# Check for unprotected JSON.parse() calls in configuration loading code
grep -r 'JSON\.parse' src/ tests/ | grep -v 'try\|catch' | wc -l | grep '^0$'

# Find fs.readFile operations on JSON files without JSON.parse protection
grep -r 'fs\.readFile.*\.json' src/ | grep -L 'JSON\.parse'

# Run integration tests for nested MCP configuration parsing
npm test -- --testNamePattern='Nested MCP propagation' --verbose
```

**Accept when:**
- All JSON.parse() invocations for configuration files are wrapped in try-catch blocks with error logging
- Integration tests for nested MCP configuration successfully parse JSON from multiple directory levels without crashes
- Grep verification shows no unprotected JSON.parse() calls in configuration loading code paths
- After successful JSON.parse(), code uses optional chaining (?.) or explicit property checks before accessing nested configuration properties

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON configuration file reads must be validated for syntactic correctness before property access. Integration tests must pass without unhandled JSON parse errors.
</enforcement>