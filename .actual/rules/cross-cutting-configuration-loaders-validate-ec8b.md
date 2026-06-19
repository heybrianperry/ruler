# Enforce JSON.parse and TOML Parsing with Error Handling for External Configuration Input: Configuration Loaders Validate

These rules are ALWAYS ACTIVE for all agent implementations and configuration loaders that process external input from files or API contracts, including OpenCodeAgent, MistralVibeAgent, CodexCliAgent, AmazonQCliAgent, and UnifiedConfigLoader.

### Rules

- **R-CONFIG-001** SHOULD: Configuration loaders SHOULD validate the structure and required fields of parsed data before returning to callers.

### Verify

```bash
# Check for unprotected JSON.parse calls in agent and core modules
grep -r 'JSON\.parse' src/agents/ src/core/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Check for unprotected parseTOML calls across the codebase
grep -r 'parseTOML' src/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Check that public API contracts with JSON.parse or parseTOML have error handling
grep -r 'api\.public\.contracts' src/agents/ | xargs grep -l 'JSON\.parse\|parseTOML' | xargs grep -L 'try.*catch' | wc -l | grep -q '^0$'
```

**Accept when:**
- All JSON.parse and parseTOML calls in agent implementations and configuration loaders are wrapped in error handling blocks (try-catch)
- No unvalidated external input is passed directly to applyRulerConfig or loadUnifiedConfig functions
- Code review confirms consistent error handling patterns across all components in policy scope
- Error messages are sanitized to avoid exposing sensitive file paths or configuration details
- Parsed and validated configurations are cached to avoid repeated validation overhead

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse and parseTOML calls must be wrapped in try-catch blocks with meaningful error context. CI pipeline MUST fail if verification commands detect unprotected parsing operations. Code review MUST block merge if input validation is missing from public API contracts.
</enforcement>