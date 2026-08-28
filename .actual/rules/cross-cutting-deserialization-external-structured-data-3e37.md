# Require Input Validation for External Structured Data Deserialization: Deserialization External Structured Data File System

These rules are ALWAYS ACTIVE for all deserialization of external structured data (JSON, TOML) from file system sources across agent implementations, MCP propagation modules, CLI handlers, and path utilities.

### Rules

- **R-DESER-001** MUST: All deserialization of external structured data from file system sources MUST be preceded by explicit validation against a defined schema or contract.
- **R-DESER-002** MUST: All deserialization operations MUST be wrapped in try-catch blocks that distinguish between parse errors (malformed syntax) and validation errors (invalid structure).
- **R-DESER-003** MUST: Validation schemas MUST be defined as typed contracts for all external structured data formats (agent configurations, MCP server definitions, CLI input, utility modules).
- **R-DESER-004** MUST: Validation failures MUST be logged with sufficient context (file path, failed constraint, actual vs expected value) without exposing sensitive data in error messages.
- **R-DESER-005** SHOULD: Implement two-phase loading for agent configuration and MCP server definitions: parse to detect syntax errors, then validate to detect schema violations.
- **R-DESER-006** SHOULD: Define validation schemas colocated with consuming modules to increase visibility and maintainability.
- **R-DESER-007** SHOULD: Design schemas to allow optional fields and unknown properties where appropriate to support future-compatible extensions.

### Verify

```bash
# Discover the project's dependency manifest and identify validation/schema libraries
find . -name 'package.json' -o -name 'Pipfile' -o -name 'Cargo.toml' -o -name 'go.mod' | head -5
grep -r 'schema\|validation\|pydantic\|zod\|serde' . --include='*.lock' --include='*.txt' --include='*.toml' 2>/dev/null | head -10

# Search for deserialization operations and verify error handling
grep -rn 'json\.load\|json\.parse\|toml\.load\|serde\|deserialize' . --include='*.py' --include='*.js' --include='*.ts' --include='*.rs' 2>/dev/null | grep -v test | head -20

# Verify deserialization is wrapped in try-catch or error handling
grep -B2 -A2 'json\.load\|json\.parse\|toml\.load' . --include='*.py' --include='*.js' --include='*.ts' 2>/dev/null | grep -E 'try|except|catch|error' | head -10

# Locate validation schema definitions
find . -path ./node_modules -prune -o -name '*schema*' -type f -print 2>/dev/null | head -10

# Verify test coverage for validation error paths
grep -rn 'test.*validation\|test.*malformed\|test.*invalid' . --include='*.py' --include='*.js' --include='*.ts' 2>/dev/null | head -10
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