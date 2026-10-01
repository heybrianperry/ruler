# ConfigLoader Module Adoption for Core Configuration Management: Core Operational Modules Delegate Configuration Retrieval

These rules are ALWAYS ACTIVE for all core operational components, engine subsystems, and module boundaries interfacing with system-level configuration parameters.

### Rules

- **R-CORE-001** MUST: Core operational modules MUST delegate all configuration retrieval, parsing, and schema validation to the ConfigLoader module rather than performing direct storage or environment reads.

### Verify

```bash
# Discover and run the project static analysis and linting scripts from the repository build manifest
# Discover and run the test runner command from the repository build manifest covering core modules
```

**Accept when:**
- Static analysis confirms zero direct environment or file system configuration reads within core operational modules outside ConfigLoader.
- All unit and integration tests covering core modules pass with configuration provided through ConfigLoader.

<enforcement>
Claude Code MUST NOT skip or defer verification. Violation handling: Pull requests containing direct environment reads or file configuration parsing outside ConfigLoader are blocked by continuous integration and must be refactored before approval.
</enforcement>