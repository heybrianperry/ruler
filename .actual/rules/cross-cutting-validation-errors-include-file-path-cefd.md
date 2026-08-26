# Schema Validation for Parsed Configuration Input: Validation Errors Include File Path Specific

These rules are ALWAYS ACTIVE for all modules that read and parse configuration files from the file system, including agent implementations, MCP integration modules, CLI command handlers, and core configuration loading infrastructure.

### Rules

- **R-SCHEMA-001** SHOULD: Validation errors SHOULD include the file path and specific field that failed validation to aid debugging.

### Verify

```bash
# Discover the project's static analysis or linting configuration and execute the verification script to detect direct parsing operations without subsequent validation
grep -r "JSON\.parse\|toml\.parse" --include="*.ts" --include="*.js" | grep -v "test\|spec" | while read line; do
  file=$(echo "$line" | cut -d: -f1)
  echo "Checking $file for validation after parsing..."
  # Verify validation occurs within 5 lines after parse call
  grep -A 5 "JSON\.parse\|toml\.parse" "$file" | grep -q "validate\|schema" || echo "WARNING: No validation found in $file"
done

# Locate the project's test suite and execute tests that verify schema validation behavior
npm test -- --testPathPattern=".*schema.*validation.*" --testNamePattern="valid|invalid"

# Search the codebase for parsing function invocations and verify each is followed by schema validation
grep -r "parse(" --include="*.ts" --include="*.js" | grep -v "test\|spec" | wc -l
grep -r "validate\|schema" --include="*.ts" --include="*.js" | grep -v "test\|spec" | wc -l
```

**Accept when:**
- All configuration parsing sites identified in the codebase include schema validation before parsed data is used
- Schema validation tests pass, demonstrating that valid configurations are accepted and invalid configurations are rejected with clear error messages
- Static analysis or code review confirms no direct parsing of external configuration without validation
- Validation error messages include both the file path and the specific field that failed validation

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing must be audited for validation coverage, and error messages must include file path and field context before code is committed.
</enforcement>