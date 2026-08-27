# Adopt Zod for Runtime Validation of Deserialized External Input: Schemas Use Zod Transformation Refinement Capabilities

These rules are ALWAYS ACTIVE for all deserialization sites that parse external file content (JSON, TOML, YAML, XML) into runtime objects, including configuration loading, agent state persistence, CLI command input parsing, and MCP server message payload handling.

### Rules

- **R-ZOD-001** MAY: Schemas MAY use Zod's transformation and refinement capabilities to normalize or sanitize input during validation.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation library and its resolved version in the lock artifact
grep -r "zod" package.json yarn.lock pnpm-lock.yaml 2>/dev/null | head -20

# Discover and execute the project's static analysis or linting configuration to verify Zod schema usage at deserialization boundaries
find . -name ".eslintrc*" -o -name "eslint.config.*" -o -name "tsconfig.json" 2>/dev/null | xargs cat 2>/dev/null | grep -i "zod\|validation" || echo "(no linting rules found)"

# Discover and execute the project's test suite, filtering for validation test cases
find . -path "*/node_modules" -prune -o -name "*.test.ts" -o -name "*.spec.ts" -o -name "*.test.js" -o -name "*.spec.js" 2>/dev/null | xargs grep -l "zod\|schema\|validation" 2>/dev/null | head -10

# Audit all deserialization sites for Zod schema validation
grep -r "JSON\.parse\|parse.*toml\|parse.*yaml" --include="*.ts" --include="*.js" . 2>/dev/null | grep -v node_modules | head -20
```

**Accept when:**
- All deserialization sites identified in the codebase (agents, CLI, MCP modules, configuration loader) have Zod schema validation with explicit error handling
- Schema definitions exist for all external data structures with appropriate strictness (required fields, type constraints, enums)
- Test coverage includes validation failure cases demonstrating that malformed input is rejected with clear error messages
- Reference ConfigLoader's schema definitions as the canonical example of Zod usage patterns within this codebase

<enforcement>
Claude Code MUST NOT skip or defer verification. All new deserialization code MUST include Zod schema validation before data use. Code review MUST block merge of deserialization code lacking schema validation. Approved exceptions require explicit justification and senior engineer or security reviewer approval, documented in code comments with rationale and risk acceptance.
</enforcement>