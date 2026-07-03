# Enforce JSON.parse and TOML Parsing with Error Handling for External Configuration Input: External Toml Input

These rules are ALWAYS ACTIVE for all agent implementations and configuration loaders that process external input from files or API contracts, including OpenCodeAgent, MistralVibeAgent, CodexCliAgent, AmazonQCliAgent, UnifiedConfigLoader, and any functions that read external JSON or TOML files from filesystem or accept configuration data through public API contracts.

### Rules

- **R-TOML-001** MUST: All external TOML input MUST be parsed using parseTOML from @iarna/toml library with explicit error handling.
- **R-TOML-002** MUST: All JSON.parse calls processing external configuration input MUST be wrapped in try-catch blocks that catch SyntaxError and provide meaningful error context including file path.
- **R-TOML-003** MUST: No unvalidated external input shall be passed directly to applyRulerConfig or loadUnifiedConfig functions.
- **R-TOML-004** MUST: All public API contracts (api.public.contracts) that accept configuration data MUST validate external input before consuming it in application logic.
- **R-TOML-005** SHOULD: Create shared utility functions for common parsing patterns to reduce code duplication across agent implementations.
- **R-TOML-006** SHOULD: Sanitize error messages before logging or displaying to users; implement structured error handling that separates internal details from user-facing messages.

### Verify

```bash
# Check for unprotected JSON.parse calls in agent and core modules
grep -r 'JSON\.parse' src/agents/ src/core/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Check for unprotected parseTOML calls across all source
grep -r 'parseTOML' src/ | grep -v 'try\|catch' | wc -l | grep -q '^0$'

# Check that all public contract parsing is wrapped in error handling
grep -r 'api\.public\.contracts' src/agents/ | xargs grep -l 'JSON\.parse\|parseTOML' | xargs grep -L 'try.*catch' | wc -l | grep -q '^0$'
```

**Accept when:**
- All JSON.parse and parseTOML calls in agent implementations and configuration loaders are wrapped in error handling blocks
- No unvalidated external input is passed directly to applyRulerConfig or loadUnifiedConfig functions
- Code review confirms consistent error handling patterns across all components in policy scope
- Error messages are sanitized and do not expose sensitive file paths or configuration details

<enforcement>
Claude Code MUST NOT skip or defer verification. All three bash verification commands must pass before accepting changes to parsing logic in agent implementations, configuration loaders, or public API contracts.
</enforcement>