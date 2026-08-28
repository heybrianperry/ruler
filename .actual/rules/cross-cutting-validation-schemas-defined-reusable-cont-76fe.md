# Require Input Validation for External Structured Data Deserialization: Validation Schemas Defined Reusable Contracts Colocated

These rules are ALWAYS ACTIVE for all agent configuration loading, MCP server definition parsing, CLI command handlers, and utility modules that deserialize external structured data (JSON, TOML) from the file system.

### Rules

- **R-VAL-001** SHOULD: Validation schemas SHOULD be defined as reusable contracts colocated with the modules that consume the data.
- **R-VAL-002** MUST: All deserialization operations for external structured data MUST be wrapped in try-catch blocks that distinguish between parse errors and validation errors.
- **R-VAL-003** MUST: Validation MUST occur before deserialized data reaches application logic.
- **R-VAL-004** SHOULD: Validation failures SHOULD be logged with sufficient context (file path, failed constraint, actual vs expected value) without exposing sensitive data.
- **R-VAL-005** SHOULD: For agent configuration and MCP server definitions, consider implementing a two-phase load: parse to detect syntax errors, then validate to detect schema violations.

### Verify

```bash
# Discover the project's dependency manifest and identify validation or schema libraries
find . -name 'package.json' -o -name 'pyproject.toml' -o -name 'Cargo.toml' -o -name 'go.mod' | head -5

# Search for deserialization operations and verify error handling
grep -r "json\.load\|toml\.load\|json\.parse\|serde" --include="*.py" --include="*.rs" --include="*.js" --include="*.ts" | grep -v test | head -20

# Verify validation schemas are defined and colocated
find . -path ./node_modules -prune -o -path ./.venv -prune -o -type f \( -name "*schema*" -o -name "*validator*" \) -print | head -10

# Locate test suite and verify validation error paths are covered
find . -path ./node_modules -prune -o -path ./.venv -prune -o -type f -name "*test*" -o -name "*spec*" | grep -i valid | head -10
```

**Accept when:**
- All deserialization operations in agent, MCP, CLI, and utility modules are wrapped in error handling that catches and logs parse failures
- Validation schemas are defined for all external structured data formats and validation occurs before deserialized data reaches application logic
- Tests demonstrate that malformed input and schema violations are rejected with actionable error messages
- Parse errors and validation errors are distinguished with different error messages
- Validation failures are logged with file path and constraint details

<enforcement>
Claude Code MUST NOT skip or defer verification. All new deserialization operations MUST include validation and error handling before merge. Static analysis or linting rules MUST detect direct deserialization calls without surrounding validation logic. Security review MUST audit module boundaries for unvalidated external input.
</enforcement>