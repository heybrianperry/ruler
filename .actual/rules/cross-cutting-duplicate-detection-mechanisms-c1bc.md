# Validate JSON Input with Try-Catch Parsing in Integration Components: Duplicate Detection Mechanisms

These rules are ALWAYS ACTIVE for all integration components that parse external JSON configuration or data, including file system operations using Node.js core libraries ('fs', 'path') and configuration loading functions (loadExistingConfig, saveConfig, applyRulerConfig, handleMcpConfiguration).

### Rules

- **R-JSON-001** MUST: Wrap all JSON.parse() calls on external input with try-catch blocks, logging errors with file path and error message context.
- **R-JSON-002** SHOULD: Use duplicate detection mechanisms (e.g., seen.add(key)) to prevent processing the same configuration multiple times.
- **R-JSON-003** MUST: Provide sensible fallback values or default configurations when parsing fails, ensuring the component can continue operating in degraded mode.
- **R-JSON-004** SHOULD: Use FileSystemUtils helper module to centralize file read error handling and reduce duplication across integration components.
- **R-JSON-005** MUST: Ensure all parse failures are logged at appropriate severity levels with sufficient context to diagnose configuration issues.

### Verify

```bash
# Verify no unwrapped JSON.parse calls on external input
grep -r 'JSON\.parse' src/ | grep -v 'try' | grep -v 'test' | wc -l | awk '{if ($1 > 0) exit 1}'

# Verify FirebenderAgent has defensive parsing with error logging
grep -A5 'JSON\.parse.*existingContent' src/agents/FirebenderAgent.ts | grep -q 'console.warn.*Failed to read/parse'

# Verify configuration loading functions implement try-catch
grep -r 'loadExistingConfig\|saveConfig\|applyRulerConfig\|handleMcpConfiguration' src/ | xargs grep -l 'try.*catch'
```

**Accept when:**
- All JSON.parse operations on external input are wrapped in try-catch blocks with error logging
- Configuration loading functions (loadExistingConfig, saveConfig, applyRulerConfig, handleMcpConfiguration) handle parse failures without throwing unhandled exceptions
- Error logs include sufficient context (file path, error message) to diagnose configuration issues
- Duplicate detection mechanisms are implemented for configuration keys that may be processed multiple times
- Fallback configurations or default values are provided when parsing fails

<enforcement>
Clause Code MUST NOT skip or defer verification. All JSON.parse operations on external input require try-catch wrapping with error logging. Violations are blocking and must be remediated before merge.
</enforcement>