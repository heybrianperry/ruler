# Enforce JSON.parse and TOML Parsing with Error Handling for External Configuration Input: Parsing Errors Include

These rules are ALWAYS ACTIVE for all agent implementations and configuration loaders that process external input from files or API contracts in JSON and TOML formats.

### Rules

- **R-PARSE-001** SHOULD: Parsing errors SHOULD include contextual information such as file path, line number, and error type to aid debugging.

### Verify

```bash
# Check for unprotected JSON.parse calls in agent and core modules
grep -r 'JSON\.parse' src/agents/ src/core/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Check for unprotected parseTOML calls
grep -r 'parseTOML' src/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Check that public API contracts with parsing are wrapped in error handling
grep -r 'api\.public\.contracts' src/agents/ | xargs grep -l 'JSON\.parse\|parseTOML' | xargs grep -L 'try.*catch' | wc -l | grep -q '^0$'
```

**Accept when:**
- All JSON.parse and parseTOML calls in agent implementations and configuration loaders are wrapped in error handling blocks
- No unvalidated external input is passed directly to applyRulerConfig or loadUnifiedConfig functions
- Code review confirms consistent error handling patterns across all components in policy scope
- Error messages include contextual information (file path, line number, error type) for debugging

<enforcement>
Claude Code MUST NOT skip or defer verification. All parsing operations on external configuration input MUST be wrapped in try-catch blocks with contextual error information before merge.
</enforcement>