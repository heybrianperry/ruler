# Require Input Validation for External Structured Data Deserialization: Deserialization Operations Wrapped Error Handling That

These rules are ALWAYS ACTIVE for all deserialization operations that consume external structured data (JSON, TOML) from file system sources across agent implementations, MCP propagation modules, CLI handlers, and path utilities.

### Rules

- **R-DESER-001** MUST: Deserialization operations MUST be wrapped in error handling that catches parse failures and provides actionable error messages.
- **R-DESER-002** MUST: All agent configuration loading from external files MUST include validation and error handling before deserialized data reaches application logic.
- **R-DESER-003** MUST: All MCP server definition parsing and propagation MUST include validation and error handling before deserialized data reaches application logic.
- **R-DESER-004** MUST: All CLI command handlers that read structured data from the file system MUST include validation and error handling before deserialized data reaches application logic.
- **R-DESER-005** MUST: All utility modules that deserialize external JSON or TOML content MUST include validation and error handling before deserialized data reaches application logic.
- **R-DESER-006** MUST: Validation schemas MUST be defined as typed contracts for all external structured data formats.
- **R-DESER-007** MUST: Validation MUST occur before deserialized data reaches application logic.
- **R-DESER-008** MUST: Parse errors and validation errors MUST be distinguished in error handling with different error messages for each case.
- **R-DESER-009** MUST: Validation failures MUST be logged with sufficient context (file path, failed constraint, actual vs expected value) without exposing sensitive data.
- **R-DESER-010** SHOULD: Consider implementing a two-phase load for agent configuration and MCP server definitions: parse to detect syntax errors, then validate to detect schema violations.

### Verify

```bash
# Discover the project's dependency manifest and identify any validation or schema libraries already present
find . -name 'package.json' -o -name 'Cargo.toml' -o -name 'pyproject.toml' -o -name 'go.mod' | head -5

# Search the codebase for deserialization operations (json.loads, serde_json, json.unmarshal, etc.)
grep -r "json\.load\|serde_json\|json\.unmarshal\|toml\.load\|yaml\.load" --include="*.py" --include="*.rs" --include="*.go" --include="*.js" --include="*.ts" . 2>/dev/null | grep -v test | grep -v node_modules

# Verify each deserialization is wrapped in try-catch or error handling
grep -B2 -A2 "json\.load\|serde_json\|json\.unmarshal\|toml\.load" --include="*.py" --include="*.rs" --include="*.go" --include="*.js" --include="*.ts" . 2>/dev/null | grep -E "try|catch|except|Result|Error|unwrap" | wc -l

# Locate test suite and verify validation error paths are covered
find . -path ./node_modules -prune -o -name '*test*' -o -name '*spec*' -type f | grep -E "\.(py|rs|go|js|ts)$" | head -10

# Search for validation schema definitions
grep -r "schema\|Schema\|validate\|Validate" --include="*.py" --include="*.rs" --include="*.go" --include="*.js" --include="*.ts" . 2>/dev/null | grep -v node_modules | grep -v test | head -20
```

**Accept when:**
- All deserialization operations in agent, MCP, CLI, and utility modules are wrapped in error handling that catches and logs parse failures
- Validation schemas are defined for all external structured data formats and validation occurs before deserialized data reaches application logic
- Tests demonstrate that malformed input and schema violations are rejected with actionable error messages
- Parse errors and validation errors are distinguished with different error messages
- Validation failures are logged with sufficient context (file path, failed constraint, actual vs expected value)

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All deserialization operations consuming external structured data MUST be audited and wrapped with validation and error handling before code review approval.
</enforcement>