# Enforce JSON.parse Input Validation for External Configuration Data: Json Configuration Files

These rules are ALWAYS ACTIVE for all agent configuration processing, MCP server configuration handling, and external JSON file parsing operations within the Ruler CLI tool and its test suite.

### Rules

- **R-JSON-001** MUST: All JSON configuration files read from the filesystem MUST be parsed using JSON.parse() with explicit error handling to catch SyntaxError exceptions.
- **R-JSON-002** MUST: Wrap all fs.readFile() + JSON.parse() sequences in try-catch blocks that handle both filesystem errors and JSON SyntaxError exceptions.
- **R-JSON-003** MUST: For agent implementations, create typed interfaces for expected configuration structures and validate parsed objects against these interfaces before property access.
- **R-JSON-004** SHOULD: Extract common validation patterns into FileSystemUtils helper functions (e.g., readJsonConfig<T>(path): T) that encapsulate error handling and type validation.
- **R-JSON-005** SHOULD: In test code, use expect() assertions on parsed JSON structure (e.g., expect(config.mcpServers).toBeDefined()) to verify both parsing success and schema correctness.
- **R-JSON-006** SHOULD: Implement secondary validation for security-sensitive fields (command, args, url, headers) with allowlists and sanitization beyond JSON parsing.
- **R-JSON-007** MAY: Test fixtures with known-valid JSON content may skip explicit error handling when the test setup guarantees valid JSON structure (EXC-001).
- **R-JSON-008** MAY: Internal JSON parsing of trusted, programmatically-generated configuration may use simplified validation if the generator is covered by unit tests (EXC-002).

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
- grep-based verification confirms zero unprotected JSON.parse() calls in agent implementations

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse() calls must include explicit error handling. Code review rejection is mandatory for violations. Security team approval is required for any exceptions in security-sensitive code paths.
</enforcement>