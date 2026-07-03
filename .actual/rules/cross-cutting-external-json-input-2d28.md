# Enforce JSON.parse and TOML Parsing with Error Handling for External Configuration Input: External Json Input

These rules are ALWAYS ACTIVE for all agent implementations, configuration loaders, and any code that processes external JSON or TOML input from filesystem paths or API contracts.

### Rules

- **R-JSON-001** MUST: All external JSON input MUST be parsed using JSON.parse with explicit error handling to catch SyntaxError exceptions.
- **R-JSON-002** MUST: All TOML parsing operations MUST use @iarna/toml parseTOML function with explicit error handling for parsing exceptions.
- **R-JSON-003** MUST: Error handling blocks wrapping JSON.parse and parseTOML calls MUST include file path context in error messages.
- **R-JSON-004** MUST: No unvalidated external input shall be passed directly to applyRulerConfig or loadUnifiedConfig functions.
- **R-JSON-005** SHOULD: Create shared utility functions for common parsing patterns to reduce code duplication across agent implementations.
- **R-JSON-006** SHOULD: Document expected configuration schema for each agent type to aid in validation and error diagnosis.
- **R-JSON-007** SHOULD: Implement caching for parsed and validated configurations to avoid repeated validation overhead.
- **R-JSON-008** SHOULD: Sanitize error messages before logging or displaying to users to prevent exposure of sensitive file paths or configuration details.

### Verify

```bash
# Check for unprotected JSON.parse calls in agent and core modules
grep -r 'JSON\.parse' src/agents/ src/core/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Check for unprotected parseTOML calls
grep -r 'parseTOML' src/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Check for unprotected parsing in public API contracts
grep -r 'api\.public\.contracts' src/agents/ | xargs grep -l 'JSON\.parse\|parseTOML' | xargs grep -L 'try.*catch' | wc -l | grep -q '^0$'
```

**Accept when:**
- All JSON.parse and parseTOML calls in agent implementations and configuration loaders are wrapped in error handling blocks
- No unvalidated external input is passed directly to applyRulerConfig or loadUnifiedConfig functions
- Error handling blocks include file path context in error messages
- Code review confirms consistent error handling patterns across all components in policy scope
- All verification commands return exit code 0 (no violations found)

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse and parseTOML calls must be wrapped in try-catch blocks with proper error context before code is considered compliant.
</enforcement>