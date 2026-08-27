# Adopt Zod for Runtime Validation of Deserialized External Input: Zod Schemas Define Expected Structure Types

These rules are ALWAYS ACTIVE for all code that deserializes external file content (JSON, TOML, YAML, XML) into runtime objects, including configuration loading, agent state persistence, CLI input parsing, and MCP integration.

### Rules

- **R-ZOD-001** MUST: Zod schemas MUST define the expected structure, types, and constraints for all deserialized data, including required fields, optional fields, enums, and nested objects.
- **R-ZOD-002** MUST: All file I/O operations that deserialize structured text formats into runtime objects MUST include Zod schema validation at the deserialization boundary, before data is used or stored in application state.
- **R-ZOD-003** MUST: Configuration loading from user-editable files or environment sources MUST validate against Zod schemas with explicit error handling that prevents invalid data from entering the application.
- **R-ZOD-004** MUST: Agent state persistence and restoration from external storage MUST validate deserialized data against Zod schemas before constructing agent instances or updating state.
- **R-ZOD-005** MUST: CLI command input parsing beyond simple string arguments MUST validate against Zod schemas.
- **R-ZOD-006** MUST: MCP server configuration and message payload deserialization MUST validate against Zod schemas.
- **R-ZOD-007** SHOULD: Schema definitions should default to strict mode and explicitly document why fields are optional or why catchall patterns are necessary.
- **R-ZOD-008** SHOULD: Reference ConfigLoader's schema definitions as the canonical example of Zod usage patterns within this codebase, including object schemas, enum validation, optional fields, and nested structures.
- **R-ZOD-009** MAY: Exceptions to schema validation require explicit justification documenting why the input source is trusted and validation is unnecessary, must be approved by a senior engineer or security reviewer, and must be documented in code comments with rationale and risk acceptance.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation library and its resolved version
find . -name 'package.json' -o -name 'package-lock.json' -o -name 'yarn.lock' -o -name 'pnpm-lock.yaml' | head -5

# Identify resolved Zod version in lock artifact
grep -A 2 '"zod"' package-lock.json 2>/dev/null || grep 'zod' yarn.lock 2>/dev/null || grep 'zod' pnpm-lock.yaml 2>/dev/null

# Discover and execute the project's static analysis or linting configuration
find . -name '.eslintrc*' -o -name 'eslint.config.*' -o -name 'biome.json' | head -3

# Discover and execute the project's test suite, filtering for validation test cases
find . -name '*.test.ts' -o -name '*.spec.ts' | xargs grep -l 'zod\|schema\|validation' 2>/dev/null | head -10

# Search for deserialization sites without schema validation
grep -r 'JSON\.parse\|parse.*toml\|parse.*yaml' --include='*.ts' --include='*.js' | grep -v 'node_modules' | grep -v '.test.ts'

# Verify Zod schema definitions exist for external data structures
find . -path ./node_modules -prune -o -name '*.ts' -type f -exec grep -l 'z\.object\|z\.enum\|z\.array' {} \; | head -20
```

**Accept when:**
- All deserialization sites identified in the codebase (agents, CLI, MCP modules, configuration loader) have Zod schema validation with explicit error handling
- Schema definitions exist for all external data structures with appropriate strictness (required fields, type constraints, enums)
- Test coverage includes validation failure cases demonstrating that malformed input is rejected with clear error messages
- Static analysis or linting rules detect raw JSON.parse or similar parse calls without subsequent validation
- Code review verifies that all new deserialization code includes Zod schema validation before data use

<enforcement>
Claude Code MUST NOT skip or defer verification. All deserialization code MUST include Zod schema validation. Code review MUST block merge of deserialization code lacking schema validation. Runtime validation failures MUST log errors and prevent invalid data from entering application state.
</enforcement>