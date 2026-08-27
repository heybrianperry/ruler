# Adopt Zod for Runtime Validation of Deserialized External Input: External Input Deserialized Files Network Responses

These rules are ALWAYS ACTIVE for all code that deserializes external input from files, network responses, or user-provided text into runtime objects, including configuration loading, agent state persistence, CLI command parsing, and MCP integration.

### Rules

- **R-ZOD-001** MUST: All external input deserialized from files, network responses, or user-provided text MUST be validated against a Zod schema before use in application logic.
- **R-ZOD-002** MUST: Schema definitions MUST exist for all external data structures with appropriate strictness, including required fields, type constraints, and enums.
- **R-ZOD-003** MUST: Validation failures MUST be handled with explicit error handling that prevents invalid data from entering application state.
- **R-ZOD-004** SHOULD: Reference ConfigLoader's schema definitions as the canonical example of Zod usage patterns, including object schemas, enum validation, optional fields, and nested structures.
- **R-ZOD-005** SHOULD: When adding validation to existing parsing sites, preserve error handling behavior while adding schema validation before the existing error boundary.
- **R-ZOD-006** MAY: Exceptions to validation requirements require explicit justification documenting why the input source is trusted and validation is unnecessary, and MUST be approved by a senior engineer or security reviewer.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation library and its resolved version
find . -name 'package.json' -o -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'pnpm-lock.yaml' | head -5

# Discover and execute the project's static analysis or linting configuration
find . -name '.eslintrc*' -o -name 'biome.json' -o -name 'rome.json' | head -5

# Discover and execute the project's test suite, filtering for validation test cases
find . -name '*.test.ts' -o -name '*.spec.ts' | xargs grep -l 'zod\|schema\|validation' | head -10

# Search for deserialization sites that may lack validation
grep -r 'JSON\.parse\|parse.*TOML\|parse.*YAML' --include='*.ts' --include='*.js' | grep -v 'node_modules' | head -20
```

**Accept when:**
- All deserialization sites identified in the codebase (agents, CLI, MCP modules, configuration loader) have Zod schema validation with explicit error handling.
- Schema definitions exist for all external data structures with appropriate strictness (required fields, type constraints, enums).
- Test coverage includes validation failure cases demonstrating that malformed input is rejected with clear error messages.
- Code review verifies that all new deserialization code includes Zod schema validation before data use.
- Static analysis or linting rules detect raw JSON.parse or similar parse calls without subsequent validation.
- Integration tests exercise validation boundaries with both valid and invalid input samples.

<enforcement>
Claude Code MUST NOT skip or defer verification. All deserialization code MUST include Zod schema validation. Code review MUST block merge of deserialization code lacking schema validation. Runtime validation failures MUST log errors and prevent invalid data from entering application state.
</enforcement>