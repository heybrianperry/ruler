# Schema Validation for Parsed Configuration Input: Schema Definitions Specify Expected Types Required

These rules are ALWAYS ACTIVE for all modules that read and parse configuration files from the file system, including agent implementations, MCP integration modules, CLI command handlers, and core configuration loading infrastructure.

### Rules

- **R-SCHEMA-001** MUST: Schema definitions MUST specify expected types, required fields, optional fields, and valid value constraints for all configuration properties.

### Verify

```bash
# Discover the project's static analysis or linting configuration and execute the verification script to detect direct parsing operations without subsequent validation
grep -r "JSON\.parse\|toml\.parse" --include="*.ts" --include="*.js" | grep -v "test" | grep -v "spec"

# Locate the project's test suite and execute tests that verify schema validation behavior
npm test -- --testPathPattern="schema|validation" --verbose

# Search the codebase for parsing function invocations and verify each is followed by schema validation before the parsed data is used in application logic
grep -A 5 "JSON\.parse\|toml\.parse" --include="*.ts" --include="*.js" | grep -E "validate|schema|assert"
```

**Accept when:**
- All configuration parsing sites identified in the codebase include schema validation before parsed data is used
- Schema validation tests pass, demonstrating that valid configurations are accepted and invalid configurations are rejected with clear error messages
- Static analysis or code review confirms no direct parsing of external configuration without validation

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing operations must be audited for schema validation coverage before code is committed.
</enforcement>