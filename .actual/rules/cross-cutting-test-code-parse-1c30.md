# Enforce JSON.parse Input Validation for External Configuration Data: Test Code Parse

These rules are ALWAYS ACTIVE for all agent configuration processing, MCP server configuration handling, and external JSON file parsing operations within the Ruler CLI tool and its test suite.

### Rules

- **R-JSON-001** MUST: Test code MUST parse JSON output from agent operations using JSON.parse() to verify configuration correctness, ensuring test assertions operate on validated data structures.
- **R-JSON-002** MUST: Wrap all fs.readFile() + JSON.parse() sequences in try-catch blocks that handle both filesystem errors and JSON SyntaxError exceptions.
- **R-JSON-003** MUST: For agent implementations, create typed interfaces for expected configuration structures and validate parsed objects against these interfaces before property access.
- **R-JSON-004** MUST: In test code, use expect() assertions on parsed JSON structure (e.g., expect(config.mcpServers).toBeDefined()) to verify both parsing success and schema correctness.
- **R-JSON-005** SHOULD: Extract common validation patterns into FileSystemUtils helper functions (e.g., readJsonConfig<T>(path): T) that encapsulate error handling and type validation.
- **R-JSON-006** SHOULD: Document expected JSON schemas in agent implementation files or separate schema documentation to guide validation logic maintenance.
- **R-JSON-007** MUST: Implement secondary validation for security-sensitive fields (command, args, url, headers) with allowlists and sanitization beyond JSON.parse().
- **R-JSON-008** MUST: Sanitize error messages in production builds to avoid information disclosure while preserving debugging information in development mode.

### Verify

```bash
# Verify no unprotected JSON.parse() calls exist
grep -r 'JSON\.parse' src/agents/ tests/ | grep -v 'try\|catch' | wc -l | awk '{if ($1 > 0) exit 1}'

# Count JSON.parse calls with proper error handling
grep -r 'JSON\.parse.*await fs\.readFile' src/ tests/ | grep -v '\.test\.' | wc -l

# Run configuration and parsing related tests
npm test -- --testNamePattern='JSON|parse|config' --passWithNoTests
```

**Accept when:**
- All JSON.parse() calls in src/agents/ are wrapped in try-catch blocks or use helper functions with error handling
- Test files demonstrate validation of parsed JSON structure using expect() assertions on required keys (mcpServers, servers, context_servers)
- Agent implementations validate configuration object structure before accessing nested properties or applying transformations
- Security-sensitive fields (command, args, url) undergo secondary validation beyond JSON parsing
- Error messages are sanitized to prevent information disclosure in production builds

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse() calls must include error handling. Code review rejection is mandatory for new JSON.parse() calls without error handling. CI pipeline failure occurs if grep-based verification detects unprotected JSON.parse() calls. Security review escalation is required for violations in command execution or network request paths.
</enforcement>