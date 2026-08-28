# Require Input Validation for External Structured Data Deserialization: Error Messages Validation Failures Indicate Which

These rules are ALWAYS ACTIVE for all deserialization operations in agent configuration loading, MCP server definition parsing, CLI command handlers, and utility modules that parse external structured data (JSON, TOML) from the file system.

### Rules

- **R-VALIDATION-001** SHOULD: Error messages from validation failures SHOULD indicate which field or constraint failed and provide guidance for correction.

### Verify

```bash
# Discover the project's dependency manifest and identify any validation or schema libraries already present in the dependency graph
find . -name 'Cargo.toml' -o -name 'pyproject.toml' -o -name 'package.json' | head -5
grep -E '(serde|pydantic|jsonschema|validator)' Cargo.toml pyproject.toml package.json 2>/dev/null || echo "No validation libraries found in manifest"

# Search the codebase for deserialization operations and verify each is wrapped in error handling and preceded by validation logic
grep -r 'serde_json::from_str\|json\.loads\|toml\.loads\|from_str' --include='*.rs' --include='*.py' --include='*.ts' --include='*.js' | grep -v test | head -20

# Locate the project's test suite and verify that validation error paths are covered by tests that assert correct error messages and rejection behavior
find . -path '*/test*' -name '*.rs' -o -path '*/test*' -name '*.py' | grep -i 'validat\|deserial' | head -10
```

**Accept when:**
- All deserialization operations in agent, MCP, CLI, and utility modules are wrapped in error handling that catches and logs parse failures
- Validation schemas are defined for all external structured data formats and validation occurs before deserialized data reaches application logic
- Tests demonstrate that malformed input and schema violations are rejected with actionable error messages that indicate which field or constraint failed
- Error messages provide specific guidance for correction rather than generic parse errors

<enforcement>
Claude Code MUST NOT skip or defer verification. All deserialization operations must be audited for validation presence and error message quality before accepting changes.
</enforcement>