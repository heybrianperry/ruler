# Adopt Zod for Runtime Validation of Deserialized External Input: Validation Failures Handled Explicitly Error Logging

These rules are ALWAYS ACTIVE for all files that deserialize external file content (JSON, TOML, YAML, XML) into runtime objects, including agent implementations, CLI handlers, MCP integration, configuration loading, and agent state persistence.

### Rules

- **R-ZOD-001** MUST: All deserialization of external file content into runtime objects MUST include Zod schema validation before the deserialized data is used or stored in application state.
- **R-ZOD-002** MUST: Validation failures MUST be handled explicitly with error logging or user-facing error messages, preventing invalid data from propagating into application state.
- **R-ZOD-003** MUST: Schema definitions MUST default to strict mode with required fields explicitly declared; optional fields and catchall patterns MUST be explicitly justified in code comments.
- **R-ZOD-004** SHOULD: Schema definitions SHOULD reference ConfigLoader's Zod patterns as the canonical example, including object schemas, enum validation, optional fields, and nested structures.
- **R-ZOD-005** SHOULD: When adding validation to existing parsing sites, validation SHOULD be inserted before the existing error boundary while preserving established error handling behavior.
- **R-ZOD-006** MAY: Exceptions to validation requirements MAY be granted only with explicit justification documenting why the input source is trusted, approved by a senior engineer or security reviewer, and documented in code comments with risk acceptance.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation library and its resolved version in the lock artifact
find . -name 'package.json' -o -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'pnpm-lock.yaml' | head -5
grep -E '"zod"|zod' package.json || echo "Zod dependency not found in manifest"

# Discover and execute the project's static analysis or linting configuration to verify Zod schema usage at deserialization boundaries
find . -name '.eslintrc*' -o -name 'eslint.config.*' | head -3

# Discover and execute the project's test suite, filtering for validation test cases
find . -name '*.test.ts' -o -name '*.spec.ts' | xargs grep -l 'zod\|schema\|validation' | head -10
```

**Accept when:**
- All deserialization sites identified in the codebase (agents, CLI, MCP modules, configuration loader) have Zod schema validation with explicit error handling
- Schema definitions exist for all external data structures with appropriate strictness (required fields, type constraints, enums)
- Test coverage includes validation failure cases demonstrating that malformed input is rejected with clear error messages
- No raw JSON.parse or TOML parse calls exist without subsequent Zod validation
- Validation failures log errors and prevent invalid data from entering application state

<enforcement>
Claude Code MUST NOT skip or defer verification. All deserialization code MUST include Zod schema validation before use. Code review MUST block merge of deserialization code lacking schema validation. Validation gaps discovered during static analysis or testing MUST be remediated before acceptance.
</enforcement>