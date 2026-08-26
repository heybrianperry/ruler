# Schema Validation for Parsed Configuration Input: Schemas Define Transformation Logic Coerce Input

These rules are ALWAYS ACTIVE for all modules that read and parse configuration files from the file system, including agent implementations, MCP integration modules, CLI command handlers, and core configuration loading infrastructure that deserialize external JSON or TOML data into runtime objects.

### Rules

- **R-SCHEMA-001** MAY: Schemas MAY define transformation logic to coerce input types or apply default values for optional fields.

### Verify

```bash
# Discover the project's static analysis or linting configuration and execute the verification script to detect direct parsing operations without subsequent validation
grep -r "JSON\.parse\|toml\.parse" --include="*.ts" --include="*.js" | grep -v "test\|spec" | while read line; do
  file=$(echo "$line" | cut -d: -f1)
  echo "Checking $file for schema validation..."
  if ! grep -A 10 "JSON\.parse\|toml\.parse" "$file" | grep -q "validate\|schema\|Schema"; then
    echo "WARNING: $file may contain unvalidated parsing"
  fi
done

# Locate the project's test suite and execute tests that verify schema validation behavior
npm test -- --testPathPattern="schema|validation" --testNamePattern="valid|invalid|coerce|transform"

# Search the codebase for parsing function invocations and verify each is followed by schema validation
grep -r "parse(" --include="*.ts" --include="*.js" | grep -v "test\|spec" | wc -l
```

**Accept when:**
- All configuration parsing sites identified in the codebase include schema validation before parsed data is used
- Schema validation tests pass, demonstrating that valid configurations are accepted and invalid configurations are rejected with clear error messages
- Static analysis or code review confirms no direct parsing of external configuration without validation
- Schemas define transformation logic for type coercion and default value application where applicable

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing must be validated through schemas before use in application logic.
</enforcement>