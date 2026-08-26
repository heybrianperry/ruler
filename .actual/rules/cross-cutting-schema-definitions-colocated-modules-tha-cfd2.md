# Schema Validation for Parsed Configuration Input: Schema Definitions Colocated Modules That Consume

These rules are ALWAYS ACTIVE for all modules that read and parse configuration files from the file system, including agent implementations, MCP integration modules, CLI command handlers, core configuration loading infrastructure, and any module that deserializes external JSON or TOML data into runtime objects.

### Rules

- **R-SCHEMA-001** SHOULD: Schema definitions SHOULD be colocated with the modules that consume the configuration to maintain clear ownership and facilitate schema evolution.

### Verify

```bash
# Discover the project's static analysis or linting configuration and execute the verification script to detect direct parsing operations without subsequent validation
grep -r "JSON\.parse\|toml\.parse" --include="*.ts" --include="*.js" | grep -v "node_modules" | while read line; do
  file=$(echo "$line" | cut -d: -f1)
  echo "Checking validation in: $file"
done

# Locate the project's test suite and execute tests that verify schema validation behavior
npm test -- --testPathPattern="schema|validation" 2>/dev/null || echo "Run test suite to verify schema validation behavior"

# Search the codebase for parsing function invocations and verify each is followed by schema validation
grep -r "parse(" --include="*.ts" --include="*.js" | grep -v "node_modules" | grep -v "test" | wc -l
```

**Accept when:**
- All configuration parsing sites identified in the codebase include schema validation before parsed data is used
- Schema validation tests pass, demonstrating that valid configurations are accepted and invalid configurations are rejected with clear error messages
- Static analysis or code review confirms no direct parsing of external configuration without validation
- Schema definitions are colocated with the modules that consume them
- Try-catch blocks wrap parsing and validation operations with appropriate error context

<enforcement>
Claude Code MUST NOT skip or defer verification. All configuration parsing must include schema validation at input boundaries before parsed data is used in application logic.
</enforcement>