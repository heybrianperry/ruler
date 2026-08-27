# Adopt Zod for Runtime Validation of Deserialized External Input: Zod Schemas Colocated Modules That Consume

These rules are ALWAYS ACTIVE for all files that deserialize external file content (JSON, TOML, YAML, XML) into runtime objects, including agent implementations, CLI handlers, MCP integration, configuration loading, and any external data source where structure and types cannot be statically guaranteed.

### Rules

- **R-ZOD-001** MUST: Apply Zod schema validation to all deserialized external input at the deserialization boundary before the data enters application state or is passed to consuming modules.
- **R-ZOD-002** SHOULD: Colocate Zod schemas with the modules that consume the validated data, maintaining clear ownership and discoverability.
- **R-ZOD-003** MUST: Define schemas with appropriate strictness by default, requiring explicit justification for optional fields, unknown types, or catchall patterns.
- **R-ZOD-004** MUST: Preserve or enhance existing error handling behavior when adding schema validation to existing parsing sites, ensuring validation failures surface with clear error messages.
- **R-ZOD-005** SHOULD: Reference ConfigLoader's schema definitions as the canonical example of Zod usage patterns within this codebase, including object schemas, enum validation, optional fields, and nested structures.
- **R-ZOD-006** MUST: For agent state deserialization, define schemas matching the persisted JSON structure and validate immediately after parsing, before constructing agent instances or updating state.
- **R-ZOD-007** SHOULD: Use Zod's optional and default value features to maintain backward compatibility with existing serialized data; version schemas when breaking changes are necessary.

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

# Audit all deserialization sites for Zod schema validation
grep -r 'JSON\.parse\|parse.*toml\|parse.*yaml' --include='*.ts' --include='*.js' . 2>/dev/null | grep -v node_modules | head -20

# Verify schema definitions exist for external data structures
find . -path '*/node_modules' -prune -o -type f \( -name '*.ts' -o -name '*.js' \) -print | xargs grep -l 'z\.object\|z\.enum\|z\.array' 2>/dev/null | head -15
```

**Accept when:**
- All deserialization sites identified in the codebase (agents, CLI, MCP modules, configuration loader) have Zod schema validation with explicit error handling.
- Schema definitions exist for all external data structures with appropriate strictness (required fields, type constraints, enums).
- Test coverage includes validation failure cases demonstrating that malformed input is rejected with clear error messages.
- Code review verifies that all new deserialization code includes Zod schema validation before data use.
- Static analysis or linting rules detect raw JSON.parse or TOML parse calls without subsequent validation.
- Integration tests exercise validation boundaries with both valid and invalid input samples.

<enforcement>
Claude Code MUST NOT skip or defer verification. All deserialization code MUST include Zod schema validation. Code review MUST block merge of deserialization code lacking schema validation. Approved exceptions require explicit justification and senior engineer or security reviewer approval, documented in code comments with rationale and risk acceptance.
</enforcement>