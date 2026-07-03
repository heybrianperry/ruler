# Enforce JSON.parse Input Validation for External Configuration Data: Parsed Json Configuration

These rules are ALWAYS ACTIVE for all agent configuration processing, MCP server configuration handling, and external JSON file parsing operations within the Ruler CLI tool and its test suite.

### Rules

- **R-JSON-001** MUST NOT: Parsed JSON configuration data MUST NOT be used directly in command execution (args, command fields) without validation of allowed values and sanitization of special characters.

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

<enforcement>
Clause R-JSON-001 MUST be verified during code review. All JSON.parse() calls handling external configuration data require explicit error handling. Violations in command execution or network request paths require security team escalation. Claude Code MUST NOT skip or defer verification of this rule.
</enforcement>