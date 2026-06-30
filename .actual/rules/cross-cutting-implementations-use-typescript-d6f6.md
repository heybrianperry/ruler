# Enforce JSON.parse Input Validation for External Configuration Data: Implementations Use Typescript

These rules are ALWAYS ACTIVE for all agent configuration processing, MCP server configuration handling, and external JSON file parsing operations within the Ruler CLI tool and its test suite.

### Rules

- **R-JSON-001** MUST: Wrap all `JSON.parse()` calls in try-catch blocks that handle both `SyntaxError` exceptions and filesystem errors when parsing external configuration data.
- **R-JSON-002** MUST: Validate parsed JSON configuration objects against expected structure (checking for required keys like `mcpServers`, `servers`, or `context_servers`) before accessing nested properties or applying transformations.
- **R-JSON-003** SHOULD: Create typed interfaces for expected configuration structures and validate parsed objects against these interfaces before property access in agent implementations.
- **R-JSON-004** SHOULD: Extract common validation patterns into helper functions (e.g., `readJsonConfig<T>(path): T` in FileSystemUtils) that encapsulate error handling and type validation.
- **R-JSON-005** MAY: Use TypeScript interfaces or JSON schema validation libraries to enforce configuration structure beyond basic `JSON.parse()` validation.
- **R-JSON-006** MUST: Implement secondary validation for security-sensitive fields (command, args, url, headers) with allowlists and sanitization beyond JSON parsing.
- **R-JSON-007** MUST: Sanitize error messages from JSON parsing failures in production builds to avoid information disclosure while preserving debugging information in development mode.

### Verify

```bash
# Verify no unprotected JSON.parse() calls exist in agent implementations
grep -r 'JSON\.parse' src/agents/ tests/ | grep -v 'try\|catch' | wc -l | awk '{if ($1 > 0) exit 1}'

# Count JSON.parse calls with proper error handling context
grep -r 'JSON\.parse.*await fs\.readFile' src/ tests/ | grep -v '\.test\.' | wc -l

# Run configuration and parsing related tests
npm test -- --testNamePattern='JSON|parse|config' --passWithNoTests
```

**Accept when:**
- All `JSON.parse()` calls in `src/agents/` are wrapped in try-catch blocks or use helper functions with error handling
- Test files demonstrate validation of parsed JSON structure using `expect()` assertions on required keys (`mcpServers`, `servers`, `context_servers`)
- Agent implementations validate configuration object structure before accessing nested properties or applying transformations
- Security-sensitive fields (command, args, url) undergo secondary validation beyond JSON parsing
- Error messages are sanitized in production builds to prevent information disclosure

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse() calls must include error handling and structural validation before use in configuration processing, command execution, or network requests.
</enforcement>