# Schema Validation for Parsed Configuration Input: Parsing Operations Handle Validation Failures Gracefully

These rules are ALWAYS ACTIVE for all modules that read and parse configuration files from the file system, including agent implementations, MCP integration modules, CLI command handlers, and core configuration loading infrastructure.

### Rules

- **R-SCHEMA-001** MUST: Parsing operations MUST handle validation failures gracefully with explicit error messages indicating which validation constraint failed.

### Verify

```bash
# Discover the project's static analysis or linting configuration and execute the verification script to detect direct parsing operations without subsequent validation
# Locate the project's test suite and execute tests that verify schema validation behavior, including tests for valid configuration acceptance and invalid configuration rejection
# Search the codebase for parsing function invocations and verify each is followed by schema validation before the parsed data is used in application logic
```

**Accept when:**
- All configuration parsing sites identified in the codebase include schema validation before parsed data is used
- Schema validation tests pass, demonstrating that valid configurations are accepted and invalid configurations are rejected with clear error messages
- Static analysis or code review confirms no direct parsing of external configuration without validation

<enforcement>
Code review MUST verify that all configuration parsing includes schema validation. Static analysis or linting rules MUST detect unvalidated parsing operations. Test coverage requirements MUST include schema validation paths for both valid and invalid input cases. Code review MUST reject changes that parse external configuration without schema validation. Runtime validation failures MUST log errors and prevent application startup with invalid configuration. Exceptions require explicit justification in code review and must be documented with inline comments explaining the validation exemption rationale.
</enforcement>