# Validate JSON Input with Try-Catch Parsing in Integration Components: Integration Components Handle

These rules are ALWAYS ACTIVE for all integration components that parse external JSON configuration or data, including file system operations using Node.js core libraries ('fs', 'path') and configuration loading functions (loadExistingConfig, saveConfig, applyRulerConfig, handleMcpConfiguration).

### Rules

- **R-INT-001** MUST: Integration components MUST handle file read failures and parse failures as separate error conditions with distinct logging.
- **R-INT-002** MUST: All JSON.parse() calls on external input MUST be wrapped in try-catch blocks with error logging that includes file path and error message context.
- **R-INT-003** MUST: Configuration loading functions (loadExistingConfig, saveConfig, applyRulerConfig, handleMcpConfiguration) MUST handle parse failures without throwing unhandled exceptions.
- **R-INT-004** SHOULD: Use FileSystemUtils helper module to centralize file read error handling and reduce duplication across integration components.
- **R-INT-005** SHOULD: Provide sensible fallback values or default configurations when parsing fails, ensuring the component can continue operating in degraded mode.

### Verify

```bash
# Check for unwrapped JSON.parse calls on external input (excluding tests)
grep -r 'JSON\.parse' src/ | grep -v 'try' | grep -v 'test' | wc -l | awk '{if ($1 > 0) exit 1}'

# Verify FirebenderAgent handles parse failures with logging
grep -A5 'JSON\.parse.*existingContent' src/agents/FirebenderAgent.ts | grep -q 'console.warn.*Failed to read/parse'

# Verify configuration loading functions use try-catch
grep -r 'loadExistingConfig\|saveConfig\|applyRulerConfig\|handleMcpConfiguration' src/ | xargs grep -l 'try.*catch'
```

**Accept when:**
- All JSON.parse operations on external input are wrapped in try-catch blocks with error logging
- Configuration loading functions (loadExistingConfig, saveConfig, applyRulerConfig, handleMcpConfiguration) handle parse failures without throwing unhandled exceptions
- Error logs include sufficient context (file path, error message) to diagnose configuration issues
- Integration tests verify graceful handling of malformed JSON configuration files

<enforcement>
Claude Code MUST NOT skip or defer verification. All JSON.parse operations on external input must be wrapped in try-catch blocks with distinct error handling for file read and parse failures. Pull requests with unwrapped JSON.parse on external input are blocked until defensive error handling is added.
</enforcement>