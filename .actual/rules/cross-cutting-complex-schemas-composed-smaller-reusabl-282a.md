# Adopt Zod for Runtime Validation of Deserialized External Input: Complex Schemas Composed Smaller Reusable Schema

These rules are ALWAYS ACTIVE for all files that deserialize external file content (JSON, TOML, YAML, XML) into runtime objects, including agent implementations, CLI handlers, MCP integration, configuration loading, and any external data source where structure and types cannot be statically guaranteed.

### Rules

- **R-ZOD-001** SHOULD: Complex schemas SHOULD be composed from smaller, reusable schema fragments to maintain consistency across validation boundaries.
- **R-ZOD-002** MUST: All file I/O operations that deserialize structured text formats into runtime objects MUST include Zod schema validation at the deserialization boundary before data use.
- **R-ZOD-003** MUST: Configuration loading from user-editable files or environment sources MUST validate against explicit Zod schemas.
- **R-ZOD-004** MUST: Agent state persistence and restoration from external storage MUST validate deserialized data against Zod schemas before constructing agent instances or updating state.
- **R-ZOD-005** MUST: CLI command input parsing beyond simple string arguments MUST include Zod schema validation.
- **R-ZOD-006** MUST: MCP server configuration and message payload deserialization MUST validate against Zod schemas.
- **R-ZOD-007** SHOULD: Schema definitions SHOULD default to strict mode and explicitly document why fields are optional or why catchall patterns are necessary.
- **R-ZOD-008** MUST: Validation failures MUST log errors and prevent invalid data from entering application state.
- **R-ZOD-009** MAY: Exceptions to validation requirements MAY be granted only with explicit justification documenting why the input source is trusted and validation is unnecessary, approved by a senior engineer or security reviewer, and documented in code comments with rationale and risk acceptance.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation library and its resolved version in the lock artifact
find . -name 'package.json' -o -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'pnpm-lock.yaml' | head -5
grep -E '"zod"|zod' package.json 2>/dev/null || echo "Check lock file for resolved version"

# Discover and execute the project's static analysis or linting configuration to verify Zod schema usage at deserialization boundaries
find . -name '.eslintrc*' -o -name 'eslint.config.*' | head -3

# Discover and execute the project's test suite, filtering for validation test cases
find . -name '*.test.ts' -o -name '*.spec.ts' | xargs grep -l 'zod\|schema\|validation' 2>/dev/null | head -10

# Audit deserialization sites for schema validation
grep -r 'JSON\.parse\|parse.*TOML\|parse.*YAML' --include='*.ts' --include='*.js' | grep -v 'node_modules' | head -20
```

**Accept when:**
- All deserialization sites identified in the codebase (agents, CLI, MCP modules, configuration loader) have Zod schema validation with explicit error handling
- Schema definitions exist for all external data structures with appropriate strictness (required fields, type constraints, enums)
- Test coverage includes validation failure cases demonstrating that malformed input is rejected with clear error messages
- Code review verifies that all new deserialization code includes Zod schema validation before data use
- Static analysis or linting rules detect raw JSON.parse or TOML parse calls without subsequent validation
- Integration tests exercise validation boundaries with both valid and invalid input samples

<enforcement>
Claude Code MUST NOT skip or defer verification. All deserialization code MUST include Zod schema validation. Code review MUST block merge of deserialization code lacking schema validation. Approved exceptions MUST be documented in code comments with explicit rationale and risk acceptance.
</enforcement>