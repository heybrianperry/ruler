# Use JSON.parse for Configuration File Deserialization in Test and Agent Code: Json Parse Calls

These rules are ALWAYS ACTIVE for all test suites and agent implementation code that processes JSON configuration files.

### Rules

- **R-JSON-001** MUST: JSON.parse() calls MUST be wrapped in try-catch blocks or error handling logic to gracefully handle malformed JSON input.

### Verify

```bash
# Count JSON.parse usage in test and agent code
grep -r 'JSON\.parse' tests/ src/agents/ --include='*.ts' | wc -l

# Verify no unsafe deserialization methods are used
grep -r 'eval\|Function(' tests/ src/agents/ --include='*.ts' | grep -v 'node_modules' || echo 'No unsafe deserialization found'

# Run configuration and JSON-related tests
npm test -- --testNamePattern='JSON|config|settings' --passWithNoTests
```

**Accept when:**
- All JSON configuration file deserialization in test and agent code uses JSON.parse() exclusively
- No instances of eval(), Function(), or other dynamic code execution methods are used for JSON deserialization
- Test suites successfully parse and validate JSON configuration files for all supported agents (Gemini CLI, Copilot, Cursor, OpenCode, Zed, Aider, Junie, etc.)
- Error handling is present around JSON.parse() calls to gracefully handle malformed JSON input

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All JSON.parse() calls in scope MUST be wrapped in try-catch blocks. Code review and CI pipeline enforcement are mandatory.
</enforcement>