# Require Input Validation for External Structured Data Deserialization: Modules Provide Fallback Defaults Optional Configuration

These rules are ALWAYS ACTIVE for all agent configuration loading, MCP server definition parsing, CLI command handlers, and utility modules that deserialize external JSON or TOML content from the file system.

### Rules

- **R-VALIDATION-001** MUST: Wrap all deserialization operations in try-catch blocks that distinguish between parse errors (malformed syntax) and validation errors (invalid structure).
- **R-VALIDATION-002** MUST: Define validation schemas as typed contracts for all external structured data formats before deserialization occurs.
- **R-VALIDATION-003** MUST: Validate external input against defined schemas before deserialized data reaches application logic.
- **R-VALIDATION-004** MUST: Provide actionable error messages that indicate exactly what is wrong with invalid input, including file path, failed constraint, and actual vs expected value.
- **R-VALIDATION-005** SHOULD: Implement two-phase loading for agent configuration and MCP server definitions: parse to detect syntax errors, then validate to detect schema violations.
- **R-VALIDATION-006** MAY: Modules MAY provide fallback defaults for optional configuration fields when validation passes but fields are absent.
- **R-VALIDATION-007** MUST: Log validation failures with sufficient context to enable rapid debugging without exposing sensitive data in error messages.

### Verify

```bash
# Discover the project's dependency manifest and identify validation or schema libraries
find . -name 'package.json' -o -name 'pyproject.toml' -o -name 'Cargo.toml' -o -name 'go.mod' | head -5

# Search for deserialization operations and verify error handling
grep -r 'json\.load\|json\.parse\|toml\.load\|serde\|deserialize' --include='*.py' --include='*.js' --include='*.ts' --include='*.rs' | grep -v test | head -20

# Verify each deserialization is wrapped in try-catch or error handling
grep -B2 -A2 'json\.load\|json\.parse\|toml\.load' --include='*.py' --include='*.js' --include='*.ts' | grep -E 'try|catch|except|error|Error' | wc -l

# Locate validation schemas in the codebase
find . -name '*schema*' -o -name '*validator*' | grep -v node_modules | grep -v '.git'

# Verify test coverage for validation error paths
grep -r 'test.*validation\|test.*malformed\|test.*invalid' --include='*.py' --include='*.js' --include='*.ts' --include='*.rs' | wc -l
```

**Accept when:**
- All deserialization operations in agent, MCP, CLI, and utility modules are wrapped in error handling that catches and logs parse failures
- Validation schemas are defined for all external structured data formats (JSON, TOML) and validation occurs before deserialized data reaches application logic
- Tests demonstrate that malformed input and schema violations are rejected with actionable error messages
- Error messages include file path, failed constraint, and actual vs expected values without exposing sensitive data
- Two-phase loading (parse then validate) is implemented for agent configuration and MCP server definitions

<enforcement>
Claude Code MUST NOT skip or defer verification. All deserialization operations must be audited for validation coverage before code review approval. Static analysis or linting rules MUST detect direct deserialization calls without surrounding validation logic. Security team MUST approve any exceptions to validation requirements for external input.
</enforcement>