# Schema Validation for Parsed Configuration Input: External Configuration Data Parsed Json Toml

These rules are ALWAYS ACTIVE for all modules that read and parse configuration files from the file system, including agent implementations, MCP integration modules, CLI command handlers, and core configuration loading infrastructure.

### Rules

- **R-SCHEMA-001** MUST: All external configuration data parsed from JSON or TOML files MUST be validated against an explicit schema before use in application logic.

### Verify

```bash
# Discover the project's static analysis or linting configuration and execute the verification script to detect direct parsing operations without subsequent validation
grep -r "JSON\.parse\|toml\.parse" --include="*.ts" --include="*.js" | grep -v "test\|spec" | while read line; do
  file=$(echo "$line" | cut -d: -f1)
  echo "Checking $file for schema validation..."
  # Verify that parsing is followed by schema validation before use
done

# Locate the project's test suite and execute tests that verify schema validation behavior
npm test -- --testPathPattern="schema|validation" --testNamePattern="valid|invalid"

# Search the codebase for parsing function invocations and verify each is followed by schema validation
grep -r "parse\|deserialize" --include="*.ts" --include="*.js" | grep -v "test\|spec" | grep -v "node_modules"
```

**Accept when:**
- All configuration parsing sites identified in the codebase include schema validation before parsed data is used
- Schema validation tests pass, demonstrating that valid configurations are accepted and invalid configurations are rejected with clear error messages
- Static analysis or code review confirms no direct parsing of external configuration without validation

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing operations must be audited and validated against this rule before code is committed.
</enforcement>