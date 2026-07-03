# Enforce JSON.parse Input Validation for External Configuration Data: Json Parsing Operations

These rules are ALWAYS ACTIVE for all agent configuration processing, MCP server configuration handling, and external JSON file parsing operations within the Ruler CLI tool and its test suite.

### Rules

- **R-JSON-001** SHOULD: JSON parsing operations SHOULD be isolated in dedicated functions (e.g., readGeminiSettings) that encapsulate validation logic and provide typed return values.
- **R-JSON-002** MUST: All JSON.parse() calls in agent implementations and configuration readers MUST be wrapped in try-catch blocks that handle both filesystem errors and JSON SyntaxError exceptions.
- **R-JSON-003** MUST: Agent implementations MUST validate parsed configuration object structure against typed interfaces before accessing nested properties or applying transformations.
- **R-JSON-004** SHOULD: Test code SHOULD use expect() assertions on parsed JSON structure (e.g., expect(config.mcpServers).toBeDefined()) to verify both parsing success and schema correctness.
- **R-JSON-005** MUST: Security-sensitive fields (command, args, url, headers) MUST undergo secondary validation beyond JSON parsing with allowlists and sanitization.
- **R-JSON-006** SHOULD: Common validation patterns SHOULD be extracted into FileSystemUtils helper functions (e.g., readJsonConfig<T>(path): T) that encapsulate error handling and type validation.
- **R-JSON-007** MUST: Error messages from JSON parsing failures MUST be sanitized in production builds to avoid information disclosure while preserving debugging information in development mode.

### Verify

```bash
# Verify no unprotected JSON.parse() calls exist in agent implementations
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
- No unprotected JSON.parse() calls are detected by grep-based verification
- Integration tests validate end-to-end configuration processing with invalid JSON fixtures

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse() operations must include error handling and structural validation before use in command execution or network requests.
</enforcement>