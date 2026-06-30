# Enforce JSON.parse Input Validation for External Configuration Data: Configuration File Paths

These rules are ALWAYS ACTIVE for all agent configuration processing, MCP server configuration handling, and external JSON file parsing operations within the Ruler CLI tool and its test suite.

### Rules

- **R-CONFIG-001** SHOULD: Configuration file paths SHOULD be validated as absolute paths before reading to prevent directory traversal attacks.
- **R-CONFIG-002** MUST: All JSON.parse() calls in agent implementations and configuration readers MUST be wrapped in try-catch blocks that handle both filesystem errors and JSON SyntaxError exceptions.
- **R-CONFIG-003** MUST: Agent implementations MUST validate parsed configuration objects against expected typed interfaces before accessing nested properties or applying transformations.
- **R-CONFIG-004** SHOULD: Test code SHOULD use expect() assertions on parsed JSON structure (e.g., expect(config.mcpServers).toBeDefined()) to verify both parsing success and schema correctness.
- **R-CONFIG-005** SHOULD: Security-sensitive fields (command, args, url, headers) SHOULD undergo secondary validation beyond JSON parsing with allowlists and sanitization.
- **R-CONFIG-006** MAY: Common validation patterns MAY be extracted into FileSystemUtils helper functions (e.g., readJsonConfig<T>(path): T) that encapsulate error handling and type validation.

### Verify

```bash
# Verify no unprotected JSON.parse() calls in agent implementations
grep -r 'JSON\.parse' src/agents/ tests/ | grep -v 'try\|catch' | wc -l | awk '{if ($1 > 0) exit 1}'

# Count JSON.parse calls with filesystem operations
grep -r 'JSON\.parse.*await fs\.readFile' src/ tests/ | grep -v '\.test\.' | wc -l

# Run configuration and parsing related tests
npm test -- --testNamePattern='JSON|parse|config' --passWithNoTests
```

**Accept when:**
- All JSON.parse() calls in src/agents/ are wrapped in try-catch blocks or use helper functions with error handling
- Test files demonstrate validation of parsed JSON structure using expect() assertions on required keys (mcpServers, servers, context_servers)
- Agent implementations validate configuration object structure before accessing nested properties or applying transformations
- Security-sensitive fields (command, args, url) undergo secondary validation beyond JSON parsing
- Configuration file paths are validated as absolute paths before reading
- Error messages from JSON parsing failures do not expose sensitive filesystem paths or configuration structure in production builds

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All JSON.parse() calls must include error handling, and configuration validation must occur before use in command execution or network requests.
</enforcement>