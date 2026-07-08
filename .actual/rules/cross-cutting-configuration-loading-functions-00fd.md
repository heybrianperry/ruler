# Validate JSON Input with Try-Catch Parsing in Integration Components: Configuration Loading Functions

These rules are ALWAYS ACTIVE for all integration components that parse external JSON configuration or data, including configuration loading functions (loadExistingConfig, saveConfig, applyRulerConfig, handleMcpConfiguration) and public API contracts exposed by integration agents.

### Rules

- **R-CONFIG-001** SHOULD: Configuration loading functions SHOULD provide fallback behavior or default values when parsing fails.
- **R-CONFIG-002** MUST: Wrap all JSON.parse() calls on external input with try-catch blocks, logging errors with file path and error message context.
- **R-CONFIG-003** SHOULD: Use FileSystemUtils helper module to centralize file read error handling and reduce duplication across integration components.
- **R-CONFIG-004** SHOULD: Implement duplicate detection with Set-based tracking (seen.add(key)) for configuration keys that may be processed multiple times.
- **R-CONFIG-005** MUST: Ensure all parse failures are logged at appropriate severity levels with sufficient context to diagnose configuration issues.

### Verify

```bash
# Verify no unwrapped JSON.parse calls on external input outside tests
grep -r 'JSON\.parse' src/ | grep -v 'try' | grep -v 'test' | wc -l | awk '{if ($1 > 0) exit 1}'

# Verify FirebenderAgent has defensive parsing with error logging
grep -A5 'JSON\.parse.*existingContent' src/agents/FirebenderAgent.ts | grep -q 'console.warn.*Failed to read/parse'

# Verify configuration loading functions use try-catch
grep -r 'loadExistingConfig\|saveConfig\|applyRulerConfig\|handleMcpConfiguration' src/ | xargs grep -l 'try.*catch'
```

**Accept when:**
- All JSON.parse operations on external input are wrapped in try-catch blocks with error logging
- Configuration loading functions (loadExistingConfig, saveConfig, applyRulerConfig, handleMcpConfiguration) handle parse failures without throwing unhandled exceptions
- Error logs include sufficient context (file path, error message) to diagnose configuration issues
- Integration components remain stable and operational even when external configuration files are malformed or corrupted
- Graceful degradation with fallback configurations is implemented rather than complete system failure

<enforcement>
Clause MUST NOT skip or defer verification. All JSON.parse operations on external input require try-catch wrapping with error logging. Pull requests with unwrapped JSON.parse on external input are blocked until defensive error handling is added. Existing violations are tracked as technical debt and prioritized by component criticality.
</enforcement>