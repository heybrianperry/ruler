# Schema Validation for Parsed Configuration Input: Before Implementing Schema Validation Developers Discover

These rules are ALWAYS ACTIVE for all modules that read and parse configuration files from the file system, including agent implementations, MCP integration modules, CLI command handlers, and core configuration loading infrastructure.

### Rules

- **R-SCHEMA-001** MUST: Before implementing schema validation, developers MUST discover the project's dependency lock file, resolve the exact installed version of the schema validation library, and consult that version's official documentation to verify API compatibility.

### Verify

```bash
# 1. Discover the project's static analysis or linting configuration and execute the verification script to detect direct parsing operations without subsequent validation
# 2. Locate the project's test suite and execute tests that verify schema validation behavior, including tests for valid configuration acceptance and invalid configuration rejection
# 3. Search the codebase for parsing function invocations and verify each is followed by schema validation before the parsed data is used in application logic
```

**Accept when:**
- All configuration parsing sites identified in the codebase include schema validation before parsed data is used
- Schema validation tests pass, demonstrating that valid configurations are accepted and invalid configurations are rejected with clear error messages
- Static analysis or code review confirms no direct parsing of external configuration without validation
- Developers have verified the exact installed version of the schema validation library against official documentation before writing validation code

<enforcement>
Code review MUST reject changes that parse external configuration without schema validation. Static analysis failures MUST block merge until validation is added. Claude Code MUST NOT skip or defer verification of dependency lock file resolution and version-specific API compatibility before implementing schema validation.
</enforcement>