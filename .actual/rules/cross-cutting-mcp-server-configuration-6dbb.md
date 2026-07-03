# Enforce JSON.parse Input Validation for External Configuration Data: Mcp Server Configuration

These rules are ALWAYS ACTIVE for all agent configuration processing, MCP server configuration handling, and external JSON file parsing operations within the Ruler CLI tool and its test suite.

### Rules

- **R-MCP-001** MUST: MCP server configuration parsing MUST validate the presence of required keys (mcpServers, servers, context_servers, mcp) appropriate to each agent's schema before merging or writing configuration.
- **R-MCP-002** MUST: Wrap all fs.readFile() + JSON.parse() sequences in try-catch blocks that handle both filesystem errors and JSON SyntaxError exceptions.
- **R-MCP-003** MUST: For agent implementations, create typed interfaces for expected configuration structures and validate parsed objects against these interfaces before property access.
- **R-MCP-004** SHOULD: Extract common validation patterns into FileSystemUtils helper functions (e.g., readJsonConfig<T>(path): T) that encapsulate error handling and type validation.
- **R-MCP-005** SHOULD: Implement secondary validation for security-sensitive fields (command, args, url, headers) with allowlists and sanitization beyond JSON.parse().
- **R-MCP-006** SHOULD: Sanitize error messages in production builds to avoid information disclosure while preserving debugging information in development mode.

### Verify

```bash
# Verify no unprotected JSON.parse() calls in agent implementations
grep -r 'JSON\.parse' src/agents/ tests/ | grep -v 'try\|catch' | wc -l | awk '{if ($1 > 0) exit 1}'

# Count JSON.parse calls with fs.readFile in source (non-test) files
grep -r 'JSON\.parse.*await fs\.readFile' src/ tests/ | grep -v '\.test\.' | wc -l

# Run configuration and JSON parsing related tests
npm test -- --testNamePattern='JSON|parse|config' --passWithNoTests
```

**Accept when:**
- All JSON.parse() calls in src/agents/ are wrapped in try-catch blocks or use helper functions with error handling
- Test files demonstrate validation of parsed JSON structure using expect() assertions on required keys (mcpServers, servers, context_servers)
- Agent implementations validate configuration object structure before accessing nested properties or applying transformations
- Security-sensitive fields (command, args, url) undergo secondary validation beyond JSON parsing
- No unprotected JSON.parse() calls are detected by static analysis
- All agent configuration tests pass, including malformed JSON fixture handling

<enforcement>
Clause Code MUST NOT skip or defer verification. All JSON.parse() calls in configuration processing paths require explicit error handling. Code review rejection is mandatory for violations. Security team approval is required for any exceptions in security-sensitive paths.
</enforcement>