# Use JSON.parse for Configuration File Deserialization in Test and Agent Code: Json Configuration File

These rules are ALWAYS ACTIVE for all test suites and agent implementation code that processes JSON configuration files.

### Rules

- **R-JSON-001** MUST: All JSON configuration file content MUST be deserialized using JSON.parse() after reading from the filesystem.
- **R-JSON-002** MUST: Wrap all JSON.parse() calls in try-catch blocks with user-friendly error messages indicating which configuration file is malformed and providing guidance for fixing JSON syntax errors.
- **R-JSON-003** MUST: After parsing, validate that required configuration keys exist before accessing nested properties.
- **R-JSON-004** SHOULD: Create a shared utility function (e.g., parseJsonConfig(filePath)) that encapsulates file reading, JSON.parse(), error handling, and basic validation for reuse across agents.
- **R-JSON-005** MUST: Never use eval(), Function(), or other dynamic code execution methods for JSON deserialization.
- **R-JSON-006** SHOULD: Implement schema validation immediately after JSON.parse() using TypeScript type guards or validation libraries (e.g., zod, ajv) to verify configuration structure before use.

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
- Configuration files are read using fs.readFile() or fs/promises readFile() with 'utf8' encoding before passing to JSON.parse()

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. All JSON configuration file deserialization MUST use JSON.parse() with proper error handling and validation.
</enforcement>