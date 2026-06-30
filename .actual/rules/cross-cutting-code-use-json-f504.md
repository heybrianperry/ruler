# Use JSON.parse for Configuration File Deserialization in Test and Agent Code: Code Use Json

These rules are ALWAYS ACTIVE for all test suites and agent implementation code that processes JSON configuration files.

### Rules

- **R-JSON-001** MUST: Use JSON.parse() for deserializing all JSON configuration files in test and agent code.
- **R-JSON-002** MUST: Wrap all JSON.parse() calls in try-catch blocks with context-specific error messages that include the configuration file path.
- **R-JSON-003** MUST: Validate that required configuration keys exist immediately after JSON.parse() before accessing nested properties.
- **R-JSON-004** MUST: Never use eval(), Function(), or other dynamic code execution methods for JSON deserialization.
- **R-JSON-005** SHOULD: Create a shared utility function (e.g., parseJsonConfig(filePath)) that encapsulates file reading, JSON.parse(), error handling, and basic validation for reuse across agents.
- **R-JSON-006** SHOULD: Read configuration files using fs.readFile() or fs/promises readFile() with 'utf8' encoding before passing content to JSON.parse().
- **R-JSON-007** MAY: Use JSON parsing libraries (e.g., json5, hjson) for enhanced JSON formats only when explicitly required by the configuration file format specification and documented in agent-specific architecture decisions.

### Verify

```bash
# Count JSON.parse usage in test and agent code
grep -r 'JSON\.parse' tests/ src/agents/ --include='*.ts' | wc -l

# Verify no unsafe deserialization methods are present
grep -r 'eval\|Function(' tests/ src/agents/ --include='*.ts' | grep -v 'node_modules' || echo 'No unsafe deserialization found'

# Run configuration and JSON-related tests
npm test -- --testNamePattern='JSON|config|settings' --passWithNoTests
```

**Accept when:**
- All JSON configuration file deserialization in test and agent code uses JSON.parse() exclusively
- No instances of eval(), Function(), or other dynamic code execution methods are used for JSON deserialization
- Test suites successfully parse and validate JSON configuration files for all supported agents (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, etc.)
- Error handling is present around JSON.parse() calls to gracefully handle malformed JSON input
- Required configuration keys are validated after parsing before use in downstream code

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON configuration deserialization MUST use JSON.parse() with proper error handling and validation. Code review MUST reject pull requests introducing eval() or Function() for JSON deserialization.
</enforcement>