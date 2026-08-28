# Require Input Validation for External Structured Data Deserialization: Validation Failures Prevent Invalid Data Reaching

These rules are ALWAYS ACTIVE for all agent configuration loading, MCP server definition parsing, CLI command handlers, and utility modules that deserialize external structured data (JSON, TOML) from the file system.

### Rules

- **R-VAL-001** MUST: Validation failures MUST prevent the invalid data from reaching application logic and MUST log the validation error with sufficient context for debugging.
- **R-VAL-002** MUST: All deserialization operations in agent, MCP, CLI, and utility modules MUST be wrapped in error handling that catches and distinguishes between parse errors (malformed syntax) and validation errors (invalid structure).
- **R-VAL-003** MUST: Validation schemas MUST be defined for all external structured data formats and validation MUST occur before deserialized data reaches application logic.
- **R-VAL-004** MUST: Validation failures MUST be logged with sufficient context (file path, failed constraint, actual vs expected value) to enable rapid debugging without exposing sensitive data in error messages.
- **R-VAL-005** SHOULD: Consider implementing a two-phase load for agent configuration and MCP server definitions: parse to detect syntax errors, then validate to detect schema violations, to provide clearer error diagnostics.
- **R-VAL-006** SHOULD: Define validation schemas as typed contracts that can be reused across modules using schema validation libraries that provide both runtime validation and static type inference.

### Verify

```bash
# Discover the project's dependency manifest and identify any validation or schema libraries already present
find . -name 'package.json' -o -name 'Cargo.toml' -o -name 'pyproject.toml' -o -name 'go.mod' | head -5

# Search the codebase for deserialization operations and verify each is wrapped in error handling
grep -r 'json\.loads\|json\.load\|toml\.loads\|toml\.load\|serde_json\|serde\|JSONDecoder\|json\.unmarshal' --include='*.py' --include='*.rs' --include='*.go' --include='*.js' --include='*.ts' | grep -v test | grep -v '.actual' | head -20

# Verify deserialization operations are preceded by validation logic
grep -B5 -A5 'json\.loads\|json\.load\|toml\.loads\|toml\.load' --include='*.py' --include='*.rs' --include='*.go' | grep -E 'validate|schema|try|except|catch|error' | head -20

# Locate the project's test suite and verify validation error paths are covered
find . -path './.actual' -prune -o -name '*test*.py' -o -name '*_test.rs' -o -name '*_test.go' -o -name '*.test.js' -o -name '*.test.ts' | grep -v '.actual' | head -10

# Verify tests assert correct error messages and rejection behavior for malformed input
grep -r 'assert.*error\|expect.*error\|should.*fail\|malformed\|invalid.*schema' --include='*test*.py' --include='*_test.rs' --include='*_test.go' --include='*.test.js' --include='*.test.ts' | head -20
```

**Accept when:**
- All deserialization operations in agent, MCP, CLI, and utility modules are wrapped in error handling that catches and logs parse failures
- Validation schemas are defined for all external structured data formats and validation occurs before deserialized data reaches application logic
- Tests demonstrate that malformed input and schema violations are rejected with actionable error messages
- Validation failures include sufficient context (file path, failed constraint, actual vs expected value) in error logs
- Parse errors and validation errors are distinguished with different error messages

<enforcement>
Claude Code MUST NOT skip or defer verification. All deserialization operations must be audited against these rules before code is committed. Security team approval is required for any exceptions to validation requirements.
</enforcement>