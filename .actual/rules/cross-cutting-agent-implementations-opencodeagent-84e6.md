# Enforce JSON.parse Input Validation for External Configuration Data: Agent Implementations Opencodeagent

These rules are ALWAYS ACTIVE for all agent implementation classes, MCP configuration processing, test harness code, and external JSON file parsing operations within the Ruler CLI tool.

### Rules

- **R-JSONPARSE-001** MUST: Agent implementations (OpenCodeAgent, ZedAgent, GeminiCliAgent) MUST validate the structure of parsed JSON objects before accessing nested properties or applying configuration transformations.
- **R-JSONPARSE-002** MUST: Wrap all fs.readFile() + JSON.parse() sequences in try-catch blocks that handle both filesystem errors and JSON SyntaxError exceptions.
- **R-JSONPARSE-003** MUST: For agent implementations, create typed interfaces for expected configuration structures and validate parsed objects against these interfaces before property access.
- **R-JSONPARSE-004** SHOULD: Extract common validation patterns into FileSystemUtils helper functions (e.g., readJsonConfig<T>(path): T) that encapsulate error handling and type validation.
- **R-JSONPARSE-005** SHOULD: Implement secondary validation for security-sensitive fields (command, args, url, headers) with allowlists and sanitization beyond JSON parsing.
- **R-JSONPARSE-006** SHOULD: Sanitize error messages in production builds to avoid information disclosure while preserving debugging information in development mode.

### Verify

```bash
# Verify no unprotected JSON.parse() calls in agent implementations
grep -r 'JSON\.parse' src/agents/ tests/ | grep -v 'try\|catch' | wc -l | awk '{if ($1 > 0) exit 1}'

# Count JSON.parse calls with file reading in non-test source files
grep -r 'JSON\.parse.*await fs\.readFile' src/ tests/ | grep -v '\.test\.' | wc -l

# Run configuration and parsing related tests
npm test -- --testNamePattern='JSON|parse|config' --passWithNoTests
```

**Accept when:**
- All JSON.parse() calls in src/agents/ are wrapped in try-catch blocks or use helper functions with error handling
- Test files demonstrate validation of parsed JSON structure using expect() assertions on required keys (mcpServers, servers, context_servers)
- Agent implementations validate configuration object structure before accessing nested properties or applying transformations
- Security-sensitive fields (command, args, url) undergo secondary validation beyond JSON parsing
- Error handling code includes both SyntaxError and filesystem error handling
- Configuration merge operations validate structure before combining agent-native and .ruler/mcp.json files

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse() calls in agent implementations and configuration processing MUST include explicit error handling and structure validation before use.
</enforcement>